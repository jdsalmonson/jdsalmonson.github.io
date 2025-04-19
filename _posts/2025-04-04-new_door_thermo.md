---
layout: post
#title: "Quantifying the Thermal Benefits of Replacement of my House\'s Front Door"
title: "Quantifying the Thermal Benefits of Replacement of the Front Door of my House"
categories: [Physics, Climate, Livermore, Electronics]
image: /assets/images/new_door_thermo/cooling_constant_distribution.png
---

<!-- [![New_Old_Door.jpeg](/assets/images/new_door_thermo/New_Old_Door.jpeg)](/assets/images/new_door_thermo/New_Old_Door.jpeg) -->
[![cooling_constant_distribution.png](/assets/images/new_door_thermo/cooling_constant_distribution.png)](/assets/images/new_door_thermo/cooling_constant_distribution.png)


## Abstract

Herein I quantify the thermal effect of replacing the front door of my house.  I acquired data with two temperature sensors I built using ESP32 microcontrollers and deployed inside and outside the house.  Two weeks of temperature data were acquired both before and after the door replacement.  The relative overnight cooling rates inside and outside the house, in the absence of artificial heat sources, are compared to Newton's Law of Cooling, i.e. the rate of indoor temperature decrease should be proportional to the temperature difference between the inside and outside of the house.  This basic analysis indicates that the cooling timescale of the house did increase with the replacement of the door, as expected.  However, this analysis also uncovers a more complex inside air temperature relaxation profile, presumably because inside air temperature is influenced both by conductive cooling from the house walls as well as from air leaking directly from the outside.  In order to address this a model is constructed that dynamically couples the indoor and outdoor air temperatures with an (unmeasured) temperature of the walls of the house.  Fitting this model to the data yields best-fit cooling constants for this system both before and after door replacement that acceptably capture the house cooling dynamics; both wall conduction and leaks of outside air.  These fits to the house cooling system indicate that replacement of the door increased the cooling time constant of the house by as much as 10%: from 26 to 28 hours, also the impact of air leaks was substantially reduced.

---

<div style="text-align: center;">
<img src="/assets/images/new_door_thermo/New_Old_Door.jpeg" width="600" alt="New_Old_Door.jpeg" />
</div>

## Data Collection

In anticipation of a long overdue replacement of the antique (estimated 1860's) front door to my house with a modern, weatherproof version, I constructed a couple basic temperature sensors.  To do this I used a couple of extra [XIAO ESP32S3](https://www.seeedstudio.com/XIAO-ESP32S3-p-5627.html?srsltid=AfmBOorm6VqDA1EE2GMdlkGD49jS8rrfBPsX4YdDZpzPoyrdKdV7dI3O) microcontrollers that I had laying around.  I've come to like the [ESP-IDF](https://github.com/espressif/esp-idf) development framework for programming the ESP32 family of chips.  The microcontroller interfaces with an MCP9808 temperature sensor over I<sup>2</sup>C and makes the data available by running a webserver on my WiFi network.  It queries the temperature sensor every second with a ```read_temperature(&current_temperature)``` call.  The webserver can receive two URI requests; one for the temperature and another for WiFi radio connection signal strength.  Each device runs a mDNS service that allows it to be found on my network with the name ```temp-sensor-1``` or ```temp-sensor-2``` respectively.

| Temperature sensors |
|:--:|
| <img src="/assets/images/new_door_thermo/temperature_sensors.jpeg" width="600" alt="temperature_sensors.jpeg" /> |
| *My identical pair of remote temperature sensors.  Each is based on a XIAO ESP32S3 microcontroller that polls the MCP9808 temperature sensor via an I<sup>2</sup>C interface and runs a web server connected to my WiFi network to which it posts the data.  The rectangular WiFi antenna sticks out to the right of each ESP32 board.* |

I collected roughly two weeks of data both before and after the door replacement.  All internal heating (i.e. the furnace) was turned off each night so the house was allowed to naturally cool overnight from roughly the same initial temperature (~ 70° Fahrenheit).  Plotting the raw indoor and outdoor temperatures over the entire duration (below) shows the diurnal oscillation period of the outdoor temperature (blue) as well as the nightly cooling curve (starting around midnight, signified by the vertical date line) of the inside temperature (red), however, beyond that, this data doesn't yield any obvious trends to my eye.  More analysis will need to be employed to see if there are underlying trends.

[![total_temperature_vs_time.png](/assets/images/new_door_thermo/total_temperature_vs_time.png)](/assets/images/new_door_thermo/total_temperature_vs_time.png)

---

### Data Analysis

#### Newton's Law of Cooling

The initial goal of this study is to model the nighttime cooling profiles with [Newton's Law of Cooling](https://en.wikipedia.org/wiki/Newton's_law_of_cooling).  Specifically, I assume that the rate of change of the house's inside temperature, $T_i$, is proportional to the difference between the outside and inside temperatures, $T_o - T_i$:

$$
\begin{equation}
\frac{dT_i}{dt} = K(T_o - T_i)
\end{equation}
$$

which is simply to say that the colder it is outside, the faster the house will cool off.  The goal is to determine the cooling constant, $K$, both before and and after door replacement to see if it is different.  In theory $K$ will be smaller after the door is replaced, meaning the house will cool more slowly for a given temperature difference $T_o - T_i$. 

The temperature history is filtered to select the nightly segments of time for which the furnace is off and the house cools naturally.  This nightly cooling data is shown in the [figures below](#cooling-data-segments).  Note that there is typically a period of relatively rapid cooling, ~1°C/hour, for roughly an hour at the beginning of the evening, when the heat is first turned off.  This is presumably due to cold outside air leaking into a porous house and replacing a portion of the warm, inside air (we will address this with the [dynamical model](#solving-a-more-complete-dynamical-cooling-model)).  Also, in the morning, after sunrise (but before the denizens have turned on the furnace) there is often a rapid decline in the cooling rate as well as the temperature difference between the inside and outside, as the outdoors begin to warm.  (Note that the time of sunrise is calculated for each day for my location in Livermore, CA.)  Both of these epochs present more complex dynamics than the simple cooling law we are interested in modeling, so we discard them both for our fits.  These appropriately filtered nightly cooling segments can now be parametrically plotted with the inside temperature cooling rate versus the temperature difference between the inside and outside, shown below.  This is the correlation we wish to fit to Newton's Law of Cooling and the respective fits for the Old and New door are plotted.  This gives the initial result that for the Old Door, $1/K = 23.6$ hours and for the new door, $1/K = 24.2$ hours.  As such it appears that the replacement of the door has increased the cooling timescale of the house by a bit over a half an hour, or about 2%, which is encouraging!  Interestingly, the residuals of these fits show an anticorrelation: the cooling rate tends to slowly decrease throughout the night faster than the difference between inside and outside temperatures would predict.  So there appears to be some cooling behavior beyond what is predicted by a simple application of Newton's Law of Cooling.

<table id="parametric-plot-linear-fit" style="width:100%; border-collapse: collapse; border: 1px solid black;">
  <tr>
    <th colspan="2" style="border: 1px solid black;">Temperature Difference vs Inside Cooling Rate</th>
  </tr>
  <tr>
    <td style="border: 1px solid black; width: 50%;">
      <a href="/assets/images/new_door_thermo/old_door_parametric_plot_linear_fit.png">
        <img src="/assets/images/new_door_thermo/old_door_parametric_plot_linear_fit.png" alt="Old Door Parametric Plot" style="width: 100%;"/>
      </a>
    </td>
    <td style="border: 1px solid black; width: 50%;">
      <a href="/assets/images/new_door_thermo/new_door_parametric_plot_linear_fit.png">
        <img src="/assets/images/new_door_thermo/new_door_parametric_plot_linear_fit.png" alt="New Door Parametric Plot" style="width: 100%;"/>
      </a>
    </td>
  </tr>
  <tr>
    <td colspan="2" style="border: 1px solid black; text-align: center;">
      <em>Parametric plots of each night's cooling segments with the Old (left) and New (right) doors.  The best fit for Newton's Law of Cooling is shown for each.  Data (dashed lines) at the beginning of each evening's cooling curve and after sunrise is excluded from the fit (see text for details).  Note a systematic anticorrelation in the residual plots: the cooling rate decreases more slowly than the temperature difference over the course of the evening.</em>
    </td>
  </tr>
</table>

There is some uncertainty in the fit of the cooling constant depending how much of the early rapid-cooling phase to include, how much data before or after sunsrise should be included and since the cooling rate, dT$_i$/dt, is a derivative which can be noisy, the amount of smoothing applied with make a difference.  For that reason I model the data over an ensemble of variations of these parameters, specifically:

- initial time to skip: 60 to 80 minutes
- end time relative to sunrise: -30 to 30 minutes
- derivative resampling window: 5 to 10 minutes

An ensemble of 200 fits are constructed for both the old and new door data, randomly varying these paramters.  

<table style="width:100%; border-collapse: collapse; border: 1px solid black;">
  <tr>
    <td style="border: 1px solid black; width: 50%;">
      <a href="/assets/images/new_door_thermo/cooling_constant_distribution.png">
        <img src="/assets/images/new_door_thermo/cooling_constant_distribution.png" alt="Cooling Constant Distribution" style="width: 100%;"/>
      </a>
    </td>
    <td style="border: 1px solid black; width: 50%;">
      <a href="/assets/images/new_door_thermo/difference_of_cooling_constants.png">
        <img src="/assets/images/new_door_thermo/difference_of_cooling_constants.png" alt="Difference of Cooling Constants" style="width: 100%;"/>
      </a>
    </td>
  </tr>
  <tr>
    <td colspan="2" style="border: 1px solid black; text-align: center;">
      <em>The distribution of fits of the cooling constant for both the Old and New door gives a distribution that is significantly longer for the Old door than for the New door (Top figure).  Plotting the difference between these cooling curves (Bottom figure) gives a mean difference of roughly 0.5 ± 0.12 hours (30 ± 7 minutes).  Note that, although I'm calculating a standard deviation, this distribution has a p-value of 0.0 and so is not normal.  This suggests that replacement of the old door has increased the 24 hour cooling time by roughly a half hour, or 2%.  This is a non-negligible increase.</em>
    </td>
  </tr>
</table>

#### Time Dependent Cooling

As previously mentioned, Newton's Cooling Law doesn't fully explain the data.  On a given night, the rate of cooling tends to decrease a little faster than the decrease in the temperature difference between the inside and outside as can be seen from the [cooling segments data](#cooling-data-segments).

$$
\begin{equation}
\frac{dT_i}{dt} = \frac{K}{1 + b \tau} (T_o - T_i)
\end{equation}
$$

where $\tau$ is the time in the cooling segment.  We fit this function from the beginning of each cooling segment, i.e. $\tau = t$, and also from the end of the segment, i.e. $\tau = t\_1 - t$ where $t\_1$ is the last time in the segment.  As with the previous time independent cooling law fit, an ensemble of 200 fits of this function are performed, varying the segment start and end time and resampling window.  The resulting histograms are plotted below.  A key observation is that when fitting this temporally decaying function from the start of the segment (so $K$ is representative of the decay constant at the beginnings of the segments) the decay constant is shorter, $1/K ~ 21$ hours, than the constant-time fit above; $1/K ~ 24$ hours.  Also, when fitting from the end of the segment, when the cooling rate tends to be lowest, the decay constant is significantly longer; $1/K ~ 29$ hours.  

The key take-away is that the observation of the longer decay constant in New door as compared to the Old door persists in these data.  Fitting the cooling time decay from the beginning of the segments gives $1/K$ of 20.8 and 21.5 hours for the Old and New doors respectively, and measuring from the end of the pulse gives 28.4 and 29.1 hours.  This is consistent with the previous result that the cooling timescale of the house was increased by roughly 30 minutes by replacing the Old door.  

The cooling rate time decay constant, $b$, doesn't exhibit an offset between the Old and New door, which isn't surprising.  It is interesting that the spread in the values of $1/b$ is noticably narrower for the New door than for the Old door.  One might hypothesize that this is due to the reduction in the Old door's drafty leaks which can cause rapid cooling, particularly early in an evening's cooling cycle.  Applying a dynamical cooling model in the next section might shed some light on this.

<table id="time-dependent-cooling-constants" style="width:100%; border-collapse: collapse; border: 1px solid black;">
  <tr>
    <td style="border: 1px solid black; text-align: center;">
      <a href="/assets/images/new_door_thermo/cooling_constant_time_dependent_fit_beginning.png">
        <img src="/assets/images/new_door_thermo/cooling_constant_time_dependent_fit_beginning.png" alt="Cooling Constant Time Dependent Fit Beginning" style="width: 100%; max-width: 1200px;"/>
      </a>
    </td>
  </tr>
  <tr>
    <td style="border: 1px solid black; text-align: center;">
      <a href="/assets/images/new_door_thermo/cooling_constant_time_dependent_fit_end.png">
        <img src="/assets/images/new_door_thermo/cooling_constant_time_dependent_fit_end.png" alt="Cooling Constant Time Dependent Fit End" style="width: 100%; max-width: 1200px;"/>
      </a>
    </td>
  </tr>
    <tr>
    <td colspan="2" style="border: 1px solid black; text-align: center;">
      <em>Time-dependent cooling constants.  The key take-away is that the cooling timescale $1/K$ for these fits shows the same difference between the Old and New door as observed in the time independent fit; the New door's cooling constant is roughly 30 minutes longer than that of the Old door.</em>
    </td>
  </tr>
</table>

---

### Solving a More Complete Dynamical Cooling Model

Upon consideration, it appears that a more elaborate process is at play.  The indoor air temperature has relatively little heat capacity compared to the house or the great outdoors and so is significantly influenced both by being in contact with the walls of the house as well as by leakage of air from the outdoors.  Unfortunately, we don't directly measure the wall temperature, but we can model the dynamical cooling of both the walls of the house as well as the indoor air temperatures as they interact with the outdoor temperature.  So we define a model where air and walls of the house are driven by their interactions with their counterparts 

$$
\begin{align}
\text{(indoor air cooling rate)} &= \text{(warming by the walls)} + \text{(leakage from outdoor air)} \\
\text{(wall cooling rate)} &= \text{(cooling by outdoor air)} + \text{(cooling by indoor air)}
\end{align}
$$

which can be written more precisely as

$$
\begin{align}
\frac{dT_i}{dt} &= K_1(T_w - T_i) + K_2(T_o - T_i) \\
\frac{dT_w}{dt} &= K_3(T_o - T_w) + K_4(T_i - T_w) \ \ .
\end{align}
$$

 The walls of the house will have signficant heat capacity and thus their temperature, $T_w$, will change the most slowly.  The indoor air temperature, $T_i$, has relatively small heat capacity and will react strongly to contact with the walls and leakage of outside air.  Our goal is to indirectly infer the cooling of the wall as well as the mixing of leaked outside air solely from the observations of the inside air temperature.

 The model is as follows: when the furnace it turned off at night it is assumed that the walls of the house and the air inside are at the same temperature, i.e. $T_w(t=0) = T_i(t=0)$.  The inside air temperature, with its relatively small heat capacity, will cool faster than the wall due to cool outside air leaking into the house via the $K_2$ term.  This relatively rapid cool down of the inside aire due to leakage with eventually be offset by the now warmer walls heating the air via the $K_1$ term.  The walls of the house, with their large heat capacity, will cool slowly by contact with the outside via the $K_3$ term, which is the dominant term of the system.  Since the heat capacity of the inside air is much, much smaller than that of the house, we can safely neglect the inside air heating the walls, so we set

 $$
 K_4 = 0 \ \ .
 $$

As such we have a three-parameter model for the cooling of the inside air, $T_i$, and house walls, $T_w$, in the presence of a measured outdoor temperature, $T_o(t)$.  Since we have also measured the inside air, $T_i(t)$, we can fit the solution, $T_{i,\text{sol}}(t)$, of this set of ODEs to the measured $T_i(t)$ and find the parameters $K_1$, $K_2$, $K_3$ that give the best match via a least-squares minimization: $\text{min}((T_{i,\text{sol}}(t) - T_i(t))^2)$.  Performing this optimization for both the Old Door and New Door data will provide information about how the cooling model of the house has changed due to the door replacement.

<table id = "parameter-optimization-results" style="width:100%; border-collapse: collapse; border: 1px solid black;">
  <tr>
    <td style="border: 1px solid black; text-align: center;">
      <a href="/assets/images/new_door_thermo/old_door_parametric_optimization_results.png">
        <img src="/assets/images/new_door_thermo/old_door_parametric_optimization_results.png" alt="Cooling Constant Time Dependent Fit Beginning" style="width: 100%; max-width: 1200px;"/>
      </a>
    </td>
  </tr>
  <tr>
    <td style="border: 1px solid black; text-align: center;">
      <a href="/assets/images/new_door_thermo/new_door_parametric_optimization_results.png">
        <img src="/assets/images/new_door_thermo/new_door_parametric_optimization_results.png" alt="Cooling Constant Time Dependent Fit End" style="width: 100%; max-width: 1200px;"/>
      </a>
    </td>
  </tr>
    <tr>
    <td colspan="2" style="border: 1px solid black; text-align: center;">
      <em>Optimization of the K<sub>1</sub>, K<sub>2</sub>, and K<sub>3</sub> parameters demonstrated here by rastering through a grid of possible values for K<sub>1</sub>, K<sub>2</sub>.  At each of these points the optimal solution for K<sub>3</sub> is iteratively found to maximize the coefficient of determination, R<sup>2</sup>.  As such we get some idea of the shape of the solution manifold.  All K constants have units of 1/hour.</em>
    </td>
  </tr>
</table>


<table style="width:100%; border-collapse: collapse; border: 1px solid black;">
  <tr>
    <th colspan="2" style="border: 1px solid black;">Dynamical Simulations of Temperature Difference vs Inside Cooling Rate</th>
  </tr>
  <tr>
    <td style="border: 1px solid black; width: 50%;">
      <a href="/assets/images/new_door_thermo/old_door_parametric_plot_dynamical_model.png">
        <img src="/assets/images/new_door_thermo/old_door_parametric_plot_dynamical_model.png" alt="Old Door Parametric Plot" style="width: 100%;"/>
      </a>
    </td>
    <td style="border: 1px solid black; width: 50%;">
      <a href="/assets/images/new_door_thermo/new_door_parametric_plot_dynamical_model.png">
        <img src="/assets/images/new_door_thermo/new_door_parametric_plot_dynamical_model.png" alt="New Door Parametric Plot" style="width: 100%;"/>
      </a>
    </td>
  </tr>
  <tr>
    <td colspan="2" style="border: 1px solid black; text-align: center;">
      <em>Similar to the <a href="#parametric-plot-linear-fit">parametric cooling figure</a> above, but instead of data, here are the solutions to the dynamical model for each evening's cooling segment.  The dashed line shows the 1/K<sub>3</sub> slope.  This illustrates how K<sub>3</sub>, that governs how fast the house walls cool due to the outside air, acts as an asymptotic limit to the cooling rate.  This is in contrast to the parameter K used to fit the mean cooling effect seen in the data, which is why the 1/K line goes thru the center of the data.  </em>
    </td>
  </tr>
</table>

The resulting best-fit solutions are:

| Parameter | Description | Old Door | New Door |
|-----------|-------------|----------|----------|
| K₁ (1/K₁) | warming of inside air by house walls | 0.8 (1.25 hours) | 0.5 (2 hours) |
| K₂ (1/K₂) | leakage of outside air to the inside | 0.07 (14.3 hours) | 0.06 (16.6 hours) |
| K₃ (1/K₃) | cooling of the house walls by oudside air | 0.0388 (25.8 hours) | 0.0356 (28.1 hours) |



<table style="width:70%; border-collapse: collapse; border: 1px solid black;">
  <tr>
    <th style="border: 1px solid black; padding: 8px;">Parameter</th>
    <th style="border: 1px solid black; padding: 8px;">Description</th>
    <th style="border: 1px solid black; padding: 8px;">Old Door</th>
    <th style="border: 1px solid black; padding: 8px;">New Door</th>
  </tr>
  <tr>
    <td style="border: 1px solid black; padding: 8px;">K₁ (1/K₁)</td>
    <td style="border: 1px solid black; padding: 8px;">warming of inside air by house walls</td>
    <td style="border: 1px solid black; padding: 8px;">0.8 (1.25 hours)</td>
    <td style="border: 1px solid black; padding: 8px;">0.5 (2 hours)</td>
  </tr>
  <tr>
    <td style="border: 1px solid black; padding: 8px;">K₂ (1/K₂)</td>
    <td style="border: 1px solid black; padding: 8px;">leakage of outside air to the inside</td>
    <td style="border: 1px solid black; padding: 8px;">0.07 (14.3 hours)</td>
    <td style="border: 1px solid black; padding: 8px;">0.06 (16.6 hours)</td>
  </tr>
  <tr>
    <td style="border: 1px solid black; padding: 8px;">K₃ (1/K₃)</td>
    <td style="border: 1px solid black; padding: 8px;">cooling of the house walls by outside air</td>
    <td style="border: 1px solid black; padding: 8px;">0.0388 (25.8 hours)</td>
    <td style="border: 1px solid black; padding: 8px;">0.0356 (28.1 hours)</td>
  </tr>
</table>

A key point to note from the <a href="#parameter-optimization-results">parameter optimization results</a> is that the optima for both the Old and New door are relatively broad, with a wide region of values with coefficients of determination, $R^2$, near 1.0.  As such, we will only interpret these results qualitatively.  However, this analysis suggests a general locus of parameters that reasonably fit the data.  

All three parameters, $K_1$, $K_2$, $K_3$ indicate weaker coupling after the New Door was installed (i.e. the inverse of each is a longer cooling timescale).  

<table style="width:50%; border-collapse: collapse; border: 1px solid black;">
  <tr>
    <td style="border: 1px solid black; text-align: center;">
      <a href="/assets/images/new_door_thermo/parametric_plot_dynamical_model_comparison.png">
        <img src="/assets/images/new_door_thermo/parametric_plot_dynamical_model_comparison.png" alt="parametric_plot_dynamical_model_comparison.png" style="max-width: 100%; height: auto;">
      </a>
      <br>
    </td>
  </tr>
  <tr>
    <td style="border: 1px solid black; text-align: center;">
      <em>For reference, the dynamical cooling solutions for both the Old and New Doors along with their respective 1/K<sub>3</sub> parameter fits.  This highlights how subtle the difference between the Old and New Door solutions is.</em>
    </td>
  </tr>
</table>

___
Reference:

- [temperature_sensor_esp32_mcp9808](https://github.com/jdsalmonson/temperature_sensor_esp32_mcp9808) - Repository of data and analysis for this notebook

---
Appendix

<!-- <a name="cooling-data-segments"></a> -->
### Cooling Data Segments
<table style="width:100%; border-collapse: collapse; border: 1px solid black;">
  <tr>
    <th colspan="2" style="border: 1px solid black;">Night-time Cooling Data Segments</th>
  </tr>
  <tr>
    <td style="border: 1px solid black; width: 50%;">
      <a href="/assets/images/new_door_thermo/old_door_segments.png">
        <img src="/assets/images/new_door_thermo/old_door_segments.png" alt="Old Door Segments" style="width: 100%;"/>
      </a>
    </td>
    <td style="border: 1px solid black; width: 50%;">
      <a href="/assets/images/new_door_thermo/new_door_segments.png">
        <img src="/assets/images/new_door_thermo/new_door_segments.png" alt="New Door Segments" style="width: 100%;"/>
      </a>
    </td>
  </tr>
  <tr>
    <td colspan="2" style="border: 1px solid black; text-align: center;">
      <em>Comparison of cooling data before (left) and after (right) door replacement.  The left plot of each pair is simply the raw temperature data from each sensor.  On the right plot of each pair, the data excluded from the fit is dashed; specifically the first 60 minutes of cooling and data after (date specific) sunrise.  The cooling (magenta) and temperature difference (black) curves are scaled by the fit to give intuition for how good it is.  A careful eye will notice that the cooling rate (magenta) tends to decrease slightly faster than the temperature difference (black) over the course of the night.</em>
    </td>
  </tr>
</table>

### Cooling Data Segments with Cooling Solutions
<table style="width:100%; border-collapse: collapse; border: 1px solid black;">
  <tr>
    <th colspan="2" style="border: 1px solid black;">Night-time Cooling Data Segments with Cooling Solutions</th>
  </tr>
  <tr>
    <td style="border: 1px solid black; width: 50%;">
      <a href="/assets/images/new_door_thermo/old_door_segments_w_solution.png">
        <img src="/assets/images/new_door_thermo/old_door_segments_w_solution.png" alt="Old Door Segments" style="width: 100%;"/>
      </a>
    </td>
    <td style="border: 1px solid black; width: 50%;">
      <a href="/assets/images/new_door_thermo/new_door_segments_w_solution.png">
        <img src="/assets/images/new_door_thermo/new_door_segments_w_solution.png" alt="New Door Segments" style="width: 100%;"/>
      </a>
    </td>
  </tr>
  <tr>
    <td colspan="2" style="border: 1px solid black; text-align: center;">
      <em>Similar to the the cooling data figures above except that the best-fit dynamical cooling solution is also plotted as solid curves.  In particular, on the left of the pair of plots the solutions for the inside air (red) and house wall (green) temperatures are shown, and on the right of the pair of plots the inside air cooling rate solution is plotted.  (Note that the house wall temperature (green) was not explicitly measured, so only the inferred solution is shown.)  Also note that (magenta) cooling rate data on the right plots is including the first 60 minutes of data (unlike the plots above), but not that after sunrise, since this dynamical fit is attempting to capture the early cooling behavior (presumably due to air leaks).  The outside temperature (blue on left plots) is measured and is an input to the dynamical solution.</em>
    </td>
  </tr>
</table>