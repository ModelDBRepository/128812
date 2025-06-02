# Grid cell network simulations. Version 1.02.

These MATLAB (R14) scripts are cleaned up versions of the scripts used to generate the manuscript figures. These will allow interested readers to verify the results or to easily continue this line of research by examining further directions. Note: Most Figure numbers were changed during revisions--while I have tried to update the figure references in these scripts, it is possible I have missed one or more.  
Eric Zilli 2010 June 1

The scripts use print_eps to save the figures as an eps file. This is available on MATLAB Central: It is really useful for making figures! (v1.02 note: print_eps is now called export_fig but is just as good)

This archive includes experimental rat trajectory information in the file rat_10925.mat. This trajectory freely available on the Moser lab website and was collected for the Hafting et al 2005 paper by members of the Moser lab. Cheers to them for making this information freely available.

The general process resulting in a 2D grid cell simulation given some model of an individual cell is:

## 1. Find injected current levels and noise level to match biological median ISI

- Variable **I** in the paper corresponds to **I** and **inputMags** in the scripts
- Noise level **sigma** in the paper corresponds to **uniqueNoiseSTD** in the scripts
- Objective: match biological median ISI of 0.2 s period (5 Hz firing rate) and a standard deviation of 0.036 s.
- Because noise causes each run to have different mean and std. dev. of the ISIs, simulate ~5000 uncoupled cells (**allncells=5000**, **pcon=0**) and check the median cell statistics.

This can be done with scripts:

* `SI_simple_model_stability_vs_params.m`
* `SI_acker_model_stability_vs_params.m`

The output of the script is a string of numbers (parameter values and measures of the network's activity) which are optionally written to a file (set **fileOut=1**). Search for **fprintf** for the meaning of the values.

After running a simulation, try (some variables may not be produced by all scripts):

- voltage variable and gating/auxiliary variables for cell #1 during entire simulation (don't forget to transpose!):
  
  ```
  figure; plot(state');
  ```

- voltage variable only:
  
  ```
  figure; plot(state(1,:));
  ```

- activity of postsynaptic cell during entire simulation
  
  ```
  figure; plot(post);
  ```

- mean ISI for each individual cell in the network
  
  ```
  figure; plot(indivmeans);
  ```

- ISI standard deviations for each individual cell in the network
  
  ```
  figure; plot(indivstds);
  ```

- estimated grid stability time for each individual cell in the network
  
  ```
  figure; plot(indivstabilities);
  ```

## 2. Find input-frequency relation (FI curve)

- This should be done to fairly high resolution.
- Guidelines to minimize simulation time:
  - Use desired spacing of the grid cell to find beta parameter:  
    `beta = sqrt(3) * spacing / 2`
  - Select maximum instantaneous velocity for accurate path integration (e.g. `v_max = 1 m/s`).
  - Frequency range needed for FI curve is then `2 * beta * v_max`.
  - Select a low frequency bound and find the input current producing it (`I_low`).
    - For Class 1 excitable cell this is essentially arbitrary.
    - For Class 2 excitable cell the cell imposes minimum firing rate.
  - Find increment `dI` moving input current about 1/200th way to desired upper frequency (`freq_high = freq_low + 2*beta*v_max`, corresponding to `I_high`).
  - Run FI simulations for `inputMags = I_low : dI : I_high`.

This can be done with scripts:

* `SI_simple_model_FI_relation.m`
* `SI_acker_model_FI_relation.m`

We include pre-generated FI curves for the cases included in the manuscript. These are:

| File                            | Description                                             |
|--------------------------------|---------------------------------------------------------|
| Acker_sn_FI_n250.mat            | Synaptically-coupled network of 250 noisy biophysical cells |
| simple_model_RS1_FI_Jan09_n1.mat| Single simple model, Type 1 excitable, regular spiking cell |
| simple_model_RS1n_FI_Jan09_n1.mat| Noisy single simple model, Type 1 excitable, regular spiking cell |
| simple_model_RS2_FI_Jan09_n1.mat| Single simple model, Type 2 excitable, regular spiking cell |
| simple_model_RS2n_FI_Jan09_n1.mat| Noisy single simple model, Type 2 excitable, regular spiking cell |
| simple_model_RS1gn_FI_Jan09_n250.mat| Gap-junction–coupled network of 250 noisy, Type 1 regular spiking simple model cells |
| simple_model_RS1sn_FI_Jan09_n250.mat| Synaptically-coupled network of 250 noisy, Type 1 regular spiking simple model cells |
| simple_model_RS2gn_FI_Jan09_n250.mat| Gap-junction–coupled network of 250 noisy, Type 2 regular spiking simple model cells |
| simple_model_RS2sn_FI_Jan09_n250.mat| Synaptically-coupled network of 250 noisy, Type 2 regular spiking simple model cells |

These files contain vectors "**currents**" and "**freqs**". `currents(i)` is the injected current level needed to drive a network to fire at `freqs(i)` Hz.

## 3. Use the FI curve in a 2D grid simulation to translate velocity signals into desired frequencies

- This allows the VCOs to be controlled.
- You still need appropriate parameters for the postsynaptic cell (**G** in the paper).
- No solved method is provided.
- Reasonable start:
  - Run 4 s simulations and compare traces of VCO cells to activity in the postsynaptic cell (like manuscript figure traces).
  - Visually decide whether stronger weights or time constants (or damping constants for resonant postsynaptic cell) need adjusting.
  - Once parameters seem successful, run longer simulations: 10 s, 40 s, 180 s making changes as needed.
  - This process can take a while.
- One analytical and one numerical technique are known but are not included.

This can be done with scripts:

* `SI_simple_model_2d_grid.m`
* `SI_acker_model_2d_grid.m`

---

The manuscript also examined firing rate adaptation. This aspect of the problem can be explored with the scripts:

* `SI_simple_model_FI_history_dependence.m`
* `SI_acker_model_FI_history_dependence.m`

---

## Parameters dependencies in scripts

- 2d_grid and FI_history_dependence scripts with single noiseless cells depend on:
  * FI_relation for frequency-current relations for single-cell VCOs.

- 2d_grid and FI_history_dependence scripts with noisy cells or networks depend on:
  * stability_vs_params for values of `uniqueNoiseSTD` (to match biology), and then `ncells`, `g`, and `pcon` to achieve high stability times despite noise.
  * FI_relation for frequency-current relations for controlling network VCOs.

- FI_relation with noisy cells or networks depends on:
  * stability_vs_params for values of `uniqueNoiseSTD`, and then `ncells`, `g`, and `pcon` to achieve high stability times despite noise.

## MANUSCRIPT ERRATA

None yet.

## CHANGELOG

### New in 1.02 (2011 Jan 10):

* Fixed plotType reference to manuscript figure numbers in comments in `SI_simple_model_2d_grid.m`.
* Fixed mistake in `hafting_trajectory.m` which was not filtering the trajectory at the same frequency as in the manuscript.
* `hafting_trajectory` no longer plots the filtered and unfiltered trajectories (uncomment those lines in `hafting_trajectory.m` if those plots are desired).
* In `SI_simple_model_2d_grid.m` simulation type comments, replaced variable **D** with **noise**.
* In `SI_simple_model_2d_grid.m` simulation type 2 has much nicer looking parameters for the resonate-and-fire model. Type 7 has nicer looking parameters for all postsynaptic models.

### New in 1.01:

* Included the trajectory from Hafting et al. 2005 which was used to make the manuscript figures.

---

2025-06-02: Converted README to Markdown.