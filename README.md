# L2/3 V1 Pyramidal Cell model (modified Petousakis et al., 2023; based on Park et al., 2019; a/n: 231185)

This folder and its subfolders contain the NEURON model used in the manuscript by Petousakis et al., 2023, as well as bash files to run the simulation. 
The model files are adapted from  Petousakis et al., 2023 (ModelDB accession number 267501).

This dataset originates from a computational study investigating the contributions of apical and basal dendrites to orientation selectivity in layer 2/3 pyramidal neurons of the mouse primary visual cortex. The data includes a detailed biophysical model, morphological reconstructions, and electrophysiological recordings, that had been validated against imaging data from previous studies. The dataset contains the computational model, shell files to run protocols, raw simulation results of voltage traces, along with analysis and plotting scripts. The simulations were done through the NEURON environment. The data was derived from the simulations of a single neuron containing 7 basal and 43 apical segments, axon and soma, passive and active mechanisms (sodium, calcium, potassium, etc. channels), with synaptic inputs modelled as Poisson spike trains, and orientation preference given to stimulus driven synapses. The dataset is reusable for studies on dendritic computation for highly detailed compartmental models.

 ## Changes made to the model

 ## Usage
 #### Compiling the model

The first step is compiling the mechanisms used by the model. Detailed instructions can be found here for:
- [Linux](https://www.neuron.yale.edu/neuron/static/docs/nmodl/unix.html)
- [Windows](https://www.neuron.yale.edu/neuron/static/docs/nmodl/mswin.html)

Briefly, you need to navigate to `./mod.files/` and open a new terminal window/command line window in that folder,
then run either "`nrnivmodl`" (linux) or "`mknrndll`" (windows).

On Windows, copy the `nrnmech.dll` file from `./mod.files/` to the main directory of the simulation files (same path as the file `L23V1_model_sim.hoc`).

On a linux-based machine, execute `run_test_sim.sh` by typing "`sh run_test_sim.sh`" in a terminal window, and a simulation of the model with the provided parameters should start.  Results will be saved in the "`results`" folder.

On a windows-based machine, execute "`run_test_sim.bat`" by double-clicking the corresponding icon, or  running it through the command line. A simulation of the model with the provided parameters should start.  Results will be saved in the "`results`" folder.

### Running the protocols

For each protocol, run the corresponding .sh files:
1.**Spontaneous activity protocol** — ongoing background excitation and inhibition with no stimulus-driven inputs.
- fig2s1btrace.sh: this is for a specific use of one of the plotting scripts that was provided. It is neuron 1, run 2 with nsB and nsA still turned off. This however we run for 5000 ms instead of 2500 (dtstopext=5000). 

2.**Orientation disparity protocol** — binocularly driven excitatory inputs with varying disparity tuning between apical and basal compartments.
- oridisp-even.sh: For the even scenario everything is split 50/50, so excdist and excdist_stim = 500, but for inhdist since 7% automatically goes to the soma the value is 46.5% (inhdist=465).
- oridisp-inverse.sh: For the inverse scenario, the excdist and excdist_stim are to be 60% apical thus = 400. For the inhibition through it is split 55.8% apical, 37.2% basal, 7% soma. And so inhdist=558.

3.**Iterative paired-pulse (I3P) protocol** — paired EPSPs delivered to dendritic branches to assess active and passive attenuation.
- i3p-nmda.sh: to test nmda spiking we turn on ttx for apical, basal, and soma (attx, bttx, sttx) to set g_na to 0 in all sections. 
- i3p-naapical.sh: to test Na spiking we need to turn on val_na in order to disable NMDA receptors. To also recieve apical activity attx will be turned off. I had ran this for all 50 segments but we only need to keep valcrits 7-49 from this.
- i3p-nabasal.sh: to test Na spiking we still have val_na on, and to get basal activity we turn attx back on but tur bttx off. From this experiment we only need to keep valcrit's 0-6.

4.**Ionic intervention protocol** — local blockade of sodium channels (ttx), inhibition removal, or stimulus suppression.
- bio-preint-sttx.sh: Run through stimulus_presentation 0-170 in steps of 10 deg. 10 neurons and 10 runs used (n_neuron, n_run). Only need soma readings, turn dend_record off. 
- even-preint-sttx.sh: Only need stimulus_presentation 0 and 90 deg, still n_neuron=10 and n_run=10, dend_record off. 
- bio-0-ionint.sh: For stimulus_presentation=0. Read in line by line the spike timings txt file (organized as: neuron run spike time), assign for each run of the simulation a line of the txt file (n_neuron=neuron, n_run=run, tpre=time-3, tpost=time+1, spikenum=spike). Spike is added into the ends of these shell files, they arent needed nessicarily but I found it easier to organize to not overwrite other files (for example if other neuron 1, run 1 has 2 spikes in that trace both spikes need to be recorded for intervention to happen on both, but not having a spike number would mean the second spike with be saved over the first one). Identifier toggles if the intervention is on the apical (-1) or basal (1) tree. Runtype turns on the intervention experiment (found in L23V1_model_sim.hoc).
- bio-90-ionint.sh: Same as above but for stimulus_presentation=90.
- even-0-ionint.sh: Same as above but stimulus_presentation=0, and background synapses are even distribution (inhdist=465, excdist=500).
- even-90-ionint.sh: Same as above but stimulus_presentation=90, and background synapses are even distribution (inhdist=465, excdist=500).

5.**Synaptic intervention protocol** — removal or silencing of specific excitatory synapses based on pre-intervention spike timing.
- synint-4a-apical.sh: For this we run through stimulus_presentations 0 to 170 deg in steps of 10 deg, 10 neurons and 10 runs (n_neuron, n_run), turning on nsA to null stim-driven synapses on the apical branch completely. 
- synint-4a-basal.sh: Same as above except we turn off nsB.
- synint-50.sh: This uses the same pre intervention as the ionic intervention (only biological condition bio-preint-sttx.sh). Read in line by line from spike timings file (organized as stim neuron run spike time), assign each run of the simulation a line of the txt file (n_neuron=neuron, n_run=run, tpre=time-30, tpost=time+10, spikenum=spike). Identifier again toggles if the intervention is on the apical or basal tree. Stmintv turns on the synaptic intervention experiment (found in L23V1_model_sim.hoc). 
- synint-nullna.sh: This is really ionic intervention but used all synint related analysis so I had put it here. For this we have identifier either -1 or 1 for apical or basal intervention, turn runtype on for ionic intervention, and read in the spike timings txt as before with tpre being time-3 and tpost is time+1. For the sensitivity analysis versions (synint-sensi-5 and synint-sensi-10.sh) it is the same as this null na file except anf is reduced to 15 and 10 respectively to lower the apical sodium conductance by 5 and 10%. A new control must be ran as well for the sensitivity analysis, rerun bio-preint-sttx.sh with the same anf changes.

6.**Apical ablation**
This is ran exactly like orientation tuning except with abltd=1. From the oridisp-bio.sh script the disparity was removed (set tag_basal to 0 deg) and then just ran the orientation tuning protocol but with the apical tree ablated. 

 ## Petousakis et al., model
modelDB: https://github.com/ModelDBRepository/267501

Konstantinos-Evangelos Petousakis Jiyoung Park Athanasia Papoutsi Stelios Smirnakis Panayiota Poirazi (2023) Modeling apical and basal tree contribution to orientation selectivity in a mouse primary visual cortex layer 2/3 pyramidal cell eLife 12:e91627.
 https://doi.org/10.7554/eLife.91627 

 ## Citation

 
