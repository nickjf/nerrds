# NERRDS | NMR Ensemble Refinement using RigiDity and Shifts v0.0.105

NERRDS uses backbone chemical shifts and a protein structure to generate an ensemble which represents conformational heterogeneity better than a traditional NMR ensemble. Our analysis (so far) suggests the conformations generated by NERRDS can reflect those adopted by proteins upon binding a drug-like molecule. More details will be available in a preprint soon. 

This method is still very much in development, please contact Nick Fowler (njfowler.com) for support, queries or suggestions.


## Installation 

`pip install nerrds`

NERRDS also requires AmberTools to be installed (specifically NERRDS requires sander or sander.MPI). The easiest way is to run NERRDS calculations on NMRbox (https://nmrbox.org). 


## Running NERRDS on a local machine (not recommended)


`nerrds -p path/to/ensemble -s path/to/shifts`


Options:

Mandatory

`-p --pdb`  | input ensemble in PDB format 

`-s --shifts` | input shifts file in NMR-STAR v3 or NEF format

Non-mandatory

`-c --conf` | number of conformers to generate per model in (default is 50, so will generate 1000 for 20 model ensemble)

`-r --avrmsd` | average RMSD of conformers generated per model in input ensemble (default 2A)

`-n --ncpu` | number of CPUs used to refine models in AMBER (default 1)

`-e --env` | path to virtual environment on NMRbox, needed to run refinement in parallel via HTCondor

`-q --quiet` | suppress output to the terminal

`-v --version` | prints version

## Running NERRDS on NMRbox (recommended)

1. Connect to NMR box

- pick a machine that is free (check https://nmrbox.org/hardware)

- connect using ssh, use -X for X-forwarding (needed if you want PANAV to check and re-reference chemical shifts): 

`ssh -X user_name@element.nmrbox.org`

2. Make a virtual environment (need to use specific version of python as the default is different across NMRbox machines, I’ve been using 3.8)

`python3.8 -m venv /path/to/new/virtual/environment` 

3. Activate virtual environment

`source /path/to/new/virtual/environment/bin/activate`

4. Install NERRDS

`pip install nerrds`

5. run NERRDS in serial (not recommended)

- Make sure to activate your virtual environment first:

`source /path/to/new/virtual/environment/bin/activate`

- run NERRDS as normal e.g. to generate 10 conformers per model, with average RMSD of 2.5 A, using 24 cpus to refine models:

`nerrds -p path/to/pdb -s path/to/shifts -c 10 -r 2.5 -n 24`

- this will run on the machine you connected to. You’ll need to stay connected for the duration of the calcs. You might need to edit your SSH settings to prevent being timed out.

6. run NERRDS with parallel refinement (recommended)

- Make sure to activate your virtual environment first:

`source /path/to/new/virtual/environment/bin/activate`

- run NERRDS using -e flag pointing to the directory where virtual environment is located e.g.

`nerrds -p path/to/pdb -s path/to/shifts -c 10 -r 2.5 -e path/to/virtual/environment`

- This will run conformer generation on the local machine then submit everything else to the HT condor queue. You can then leave the calcs to run and check back later. Note: in principle you can ask for multiple cpus but it will faster to run each in parallel with 1 cpu (default).


## Output

NERRDS saves everything in a directory named date_time_pdbfilename_shiftsfilename

Output consists of NERRDS.log logfile and 6 directories

`models` | this contains the models extracted from the input ensemble.

`anm` | this contains all the conformers generated by ProDy, numbered according to which model in the input ensemble they were generated from. Numbers start at 2 because model 1 is always the input model (as is discarded by NERRDS).

`refined` | this contains directories where models are refined. After refinement all trajectories are deleted to save space. The last frame in the trajectory is saved as a PDB. Output files &ast;.out which detail the refinement process are retained. Not all models will refine successfully. There is an inverse relationship between the number of models which refine and the average RMSD of conformers generated. Models which do refine successfully are combined into a single file named anm_pdbfile.pdb. model_link.json maps filenames to model numbers.

`ansurr` | this contains ANSURR output ran using the input shifts file and the ensemble containing successfully refined models.

`bme` | this directory is used to run Bayesian Meaximum Entropy. exp.txt is per residue rigidity according to chemical shifts and error which has been set to 0.05. calc.txt is per residue rigidity for each model. Various other files are generated which don’t take up much space so I have left for now.

`ensemble` | this directory contains the selected models which are saved so that the weight is the last number in the filename e.g. anm_2l28_bestnoe_100_A_edit_4_4_refined_4.pdb has a weight of 0.04, anm_2l28_bestnoe_100_A_edit_35_9_refined_22.pdb has a weight of 0.22. The models are also combined into a single ensemble. REMARK records are used to map models to the filenames (and therefore also the weights).

## Help

Contact Nick Fowler (njfowler.com) for support, queries or suggestions.


## Acknowledgements

NMRBox | https://nmrbox.org

ProDy | Zhang S, Krieger JM, Zhang Y, Kaya C, Kaynak B, Mikulska-Ruminska K, Doruker P, Li H, Bahar I ProDy 2.0: Increased scale and scope after 10 years of protein dynamics modelling with Python, Bioinformatics, (2021).  https://doi.org/10.1093/bioinformatics/btab187

Bayesian Maximum Entropy | Bottaro, S., Bengtsen, T., Lindorff-Larsen, K. (2020). Integrating Molecular Simulation and Experimental Data: A Bayesian/Maximum Entropy Reweighting Approach. In: Gáspári, Z. (eds) Structural Bioinformatics. Methods in Molecular Biology, vol 2112. Humana, New York, NY. https://doi.org/10.1007/978-1-0716-0270-6_15

AmberTools | D.A. Case, H.M. Aktulga, K. Belfon, I.Y. Ben-Shalom, J.T. Berryman, S.R. Brozell, D.S. Cerutti, T.E. Cheatham, III, G.A. Cisneros, V.W.D. Cruzeiro, T.A. Darden, R.E. Duke, G. Giambasu, M.K. Gilson, H. Gohlke, A.W. Goetz, R. Harris, S. Izadi, S.A. Izmailov, K. Kasavajhala, M.C. Kaymak, E. King, A. Kovalenko, T. Kurtzman, T.S. Lee, S. LeGrand, P. Li, C. Lin, J. Liu, T. Luchko, R. Luo, M. Machado, V. Man, M. Manathunga, K.M. Merz, Y. Miao, O. Mikhailovskii, G. Monard, H. Nguyen, K.A. O'Hearn, A. Onufriev, F. Pan, S. Pantano, R. Qi, A. Rahnamoun, D.R. Roe, A. Roitberg, C. Sagui, S. Schott-Verdugo, A. Shajan, J. Shen, C.L. Simmerling, N.R. Skrynnikov, J. Smith, J. Swails, R.C. Walker, J. Wang, J. Wang, H. Wei, R.M. Wolf, X. Wu, Y. Xiong, Y. Xue, D.M. York, S. Zhao, and P.A. Kollman (2022), Amber 2022, University of California, San Francisco.

ANSURR | Fowler, N.J., Sljoka, A. & Williamson, M.P. A method for validating the accuracy of NMR protein structures. Nat Commun 11, 6321 (2020). https://doi.org/10.1038/s41467-020-20177-1

Random Coil Index (RCI) | Berjanskii, M.V. & Wishart, D.S. A simple method to predict protein flexibility using secondary chemical shifts. Journal of the American Chemical Society 127, 14970-14971 (2005).

Floppy Inclusions and Rigid Substructure Topography (FIRST) | Jacobs, D.J., Rader, A.J., Kuhn, L.A. & Thorpe, M.F. Protein flexibility predictions using graph theory. Proteins-Structure Function and Genetics 44, 150-165 (2001).

Probabilistic Approach to NMR Assignment and Validation (PANAV) | Bowei Wang, Yunjun Wang and David S. Wishart. "A probabilistic approach for validating protein NMR chemical shift assignments". Journal of Biomolecular NMR. Volume 47, Number 2 / June 2010: 85-99
