SMaPPOI installation and usage notes
v2.0 (3 Jul. 2020)
J. Peter Rickgauer
rickgauerj@janelia.hhmi.org


SYSTEM REQUIREMENTS

-64-bit GNU/Linux (tested on Scientific Linux; 3.10.0-693.21.1.el7.x86_64) 
	-recommended hardware (as tested): 128 GB RAM, 10x Intel(R) Xeon(R) CPU E5-2630 v4 @ 2.20GHz
-1 or more GPU board (recommended: NVIDIA GeForce GTX1080 or equivalent; 8 GB onboard memory)
-Matlab Runtime Libraries v. 9.6 (free to download; http://www.mathworks.com/products/compiler/mcr)



INSTALLATION

1. Create a new directory (the "install directory") and extract the contents of the archive into that directory.
2. Append the install directory to the system path.
3. Install the Matlab Runtime Libraries using the following procedure (installation is required to run compiled versions of the code):
	a. Download the R2019a MCR installer zipfile from the Mathworks website (http://www.mathworks.com/products/compiler/mcr) into the "MCR/" directory within the install directory.
	b. Go to the MCR subdirectory and unzip the installer zipfile.
	c. Install the libraries by running the following from the command line, after replacing <absolutePathToPwd/> with the full absolute path to the MCR subdirectory as appropriate:

		./install -mode silent -agreeToLicense yes -destinationFolder <absolutePathToPwd/> -outputFile <absolutePathToPwd/>

	d. once installation finishes, the installer should display several lines of instructions that tell you how to change your environment variables ($LD_LIBRARY_PATH and possibly $XAPPLRESDIR). Follow those instructions, then restart the command shell (or source your .bashrc / equivalent file).
	e. If this does not work, go to https://www.mathworks.com/help/compiler/install-the-matlab-runtime.html for other ideas.
4. Test the code using sample files included in the archive as described below in "EXAMPLES".

Once the MCR installer zipfile is downloaded, installation should take less than 10 minutes.


SYNTAX

./smappoi_run.sh <program> <paramFile>

<program> can be either calcSP_v2 or searchImage_v2. <paramFile> is an ASCII file with parameters appropriate to the program being run (see below), and is parsed by the shell script launcher (smappoi_run.sh) and the program itself. smappoi_run.sh is a shell script that launches parallel instances of these programs using the requested number of GPU boards, passing along the parameters and a job ID to each instance.



EXAMPLES

In the following, "$ID" is used to represent the install directory. 

1. Calculate the electrostatic and scattering potential matrices for a known reference structure. Using 8 GPU boards (NVIDIA GeForce GTX1080; 8 GB on-board memory), this calculation should take about 5-6 minutes (~40-48 minutes with 1 GTX1080 board).

	a. Open $ID/6EK0_LSU.par and modify the line that reads "nCores 8" to match the number of available GPU boards that you wish to use in the calculation (e.g., "nCores 2"). Save and close the file.
	b. From $ID, start the calculation by running the following (note that the basename of the .par file is used as basename for the scattering potential files and logs):

	./smappoi_run.sh 6EK0_LSU.par

	c. When the calculation is complete, verify that there are 3 files in $ID/test/model_001/: 1) the electrostatic potential; 2) a padded scattering potential; and 3) a concatenated log file. 





2. Search an experimental image of a mouse embryonic fibroblast cell to determine the location and orientation of ribosomal LSUs. Using 8 GPU boards (NVIDIA GeForce GTX1080; 8 GB on-board memory), this search should take about 11-12 minutes (~90 minutes with one board).

	a. Open $ID/samples/sample_searchImage.par and modify nCores if appropriate. Modify the file paths to suit your install directory, including the model generated in Example 1 (which is required here).
	b. From $ID, start the calculation by running:

		./smappoi_run.sh sample_search_001.par

	c. When the search is complete, verify that there are 13 additional files in the search output directory (as specified in the .par file) with the following names:

	1. sample_search_001.par: copy of the .par file used in the search
	2. searchImage.mrc: the image used for cross-correlation (whitened if psdFilterFlag is set to 1)
	3. search_listAboveThreshold.dat: list of all CC values above the threshold specified by "arbThr" in the parameter file
	4. search_listAboveThreshold_header.dat: (obsolete)
	5. search_vals.mrc: maximum intensity projection (MIP) of all cross-correlation (CC) values at each pixel in the image, after flat-fielding; each section in the stack corresponds to one assumed defocus within the evenly-spaced range specified in the .par file
	6. search_qInds.mrc: indices of rotation matrices at each pixel in the MIP that produced the corresponding CC value. Multiple planes correspond to multiple assumed defocuses, as above 
	7. search_SH.txt: survival histograms of flat-fielded search values at each specified SNR value (column 1) as detected in the search (column 3) and as expected for an equal number of values drawn from a Gaussian Normal distribution (column 2)
	8. highVals.txt: list of initial (pre-flat fielded and -refined) values from the search, sorted ascending by CC value. Columns indicate: [rotation_index CC coord coord defocus_index]
	9. search_mean.mrc: mean CC values at each pixel, calculated across all CCs in the search
	10. search_SD.mrc: standard deviation (SD) of all CC values at each pixel, calculated across all CCs in the search
	11. search_SDs.dat: list of standard deviations for all templates tested in the search, prior to normalizing (i.e., setting the SD to one). Provides a sense of the variance across projections
	12. list_final.txt: list of clustered particles detected after flat-fielding, thresholding, and refinement. Columns indicate: 
		CC	coord	coord	df1	df2	ast_ang	q1		q2		q3		q4
		13.039  3291    477     500.85  379.05  0.960   0.301101        0.441231        -0.007914       -0.845335
	13. search.log: log file

Note that this search only includes a sparse subset of the rotations needed to ensure complete coverage of orientation space. A more typical search set includes 2,359,296 rotations (64x the number tested here), and may be explored by changing the .par file's "rotationsFile" parameter to "rotation/hopf_R.txt").


PARAMETER FILES

searchImage_v2 requires an input file with input parameters structured like the following example:

function $ID/searchImage_v2
psdFilterFlag 1
arbThr 5.0
optThr 7.0
optimize_flag 1
aPerPix 1.032
V_acc 300000.0
Cs 0.000001
Cc 0.0027
deltaE 0.7
a_i 0.000050
F_abs 0.0
binRange 15.0
nBins 1024
qThr 10
dThr 10
T_sample 170
df_inc 42.5
defocus 440.7 318.9 0.960
imageFile $ID/v2.0/image/061518_F_0012_search.mrc
modelFile $ID/v2.0/test/model_001/6EK0_LSU_SP.mrc
rotationsFile $ID/v2.0/rotation/hopf_R3.txt
outputDir $ID/v2.0/test/search_001-6EK0_LSU-hopf_R3
nCores 8
#any line commented with a pound sign is ignored


Descriptions [and where applicable, recommended values] follow:
function: the compiled function to run (searchImage_v2)
psdFilterFlag [1]: choose to apply a whitening filter based on the reciprocal square root of the radially averaged power spectral density of the image
arbThr [5.0 or greater]: threshold CC value above which all values are saved (with corresponding pixel coordinates and rotation matrix indices)
optimize_flag [1]: choose to refine the orientation, position, and defocus of particles found in a search.
optThr [7.0 or greater]: minimum SNR (pre-flat fielded) needed to qualify a particle (cluster) for post-search refinement
aPerPix: voxel or pitch assumed for the image and model (in Angstroms)
V_acc: microscope accelerating voltage (in V)
Cs: spherical aberration coefficient (in m)
Cc: chromatic aberration coefficient (in m)
deltaE: energy spread of the source (in eV)
a_i [0.000050]: illumination aperture (in radians)
F_abs: amplitude contrast coefficient (0-1.0)
binRange: binning range for CC histograms (in SNR units, or standard deviations above the mean for what is expected of a Gaussian noise distribution of similar size). 
nBins [1024]: (obsolete)
qThr [10]: minimum angular distance (in degrees) separating two above-threshold CC maxima before spawning a second cluster among overlapping peaks
dThr [10]: minimum euclidean distance (in Angstroms) separating two above-threshold CC maxima before spawning a second cluster among overlapping peaks
T_sample: estimated sample thickness (units: nanometers). Used with df_inc to determine the range of assumed defocus planes to search
df_inc: defocus step-size (units: nanometers) used in the global image search
defocus: astigmatic defocus (see Rohou and Grigorieff, JSB 2015)
imageFile: name of input image to search (.mrc)
modelFile: name of scattering potential volume to use for template generation (.mrc)
rotationsFile: name of file including rotation matrices specifying template orientations
outputDir: directory for output files
nCores: # of GPU boards to use for calculation


calcSP_v2 requires an input file with input parameters structured like the following example:

function $ID/v2.0/calcSP_v2
PDBFile $ID/v2.0/model/6EK0_LSU.pdb
aPerPix 1.032
V_acc 300000
outputDir $ID/v2.0/test/model_001
nCores 8


Descriptions:
function: the compiled function to run (calcSP_v2)
PDBFile: name of input .pdb file
aPerPix: voxel pitch (in Angstroms)
V_acc: microscope accelerating voltage (in V)
outputDir: directory for output files
nCores: number of GPU boards to use for calculation



FILES INCLUDED

# source code:
src/calcSP_v2.m 
src/searchImage_v2.m
src/@smap/ (private/static methods class; called by calcSP_v2 and searchImage_v2)

# shell script to run calcSP_v2 and searchImage_v2 binaries:
smappoi_run.sh

# binaries (compiled with Matlab Compiler mcc v9.6, R2019a):
calcSP_v2 # calculates elastic scattering potential matrix for a molecule, given its .pdb file
searchImage_v2 # given a scattering potential, calculates projection templates for a given set of microscope properties and cross-correlates them against an image

# sample files:
6EK0_LSU.par: sample parameters file for calcSP_v2 function
sample_search_001.par, sample_search_002.par: sample parameter files for searchImage_v2 function
image/061518_F_0012_search.mrc: experimental image for searches
model/6EK0_LSU.pdb: PDB-formatted file including the large ribosomal subunit
sample_output/* : expected logfiles from Examples 1-2 above, and a subset of expected text output files from Example 2 (search) run using two sets of rotation matrices with different sampling densities (hopf_R3.txt and hopf_R4.txt)

# rotation matrix sets:
rotation/hopf_R.txt: 2,359,296 rotation matrices generated using the hopf fibration (see REFERENCES)
rotation/hopf_R4.txt:  294,912 rotation matrices generated using the hopf fibration (see REFERENCES)
rotation/hopf_R3.txt:   36,864 rotation matrices generated using the hopf fibration (see REFERENCES)


REFERENCES

This code makes use of modified versions of the following external code:
1. ReadMRC and associated code - F. Sigworth; File ID #27021, https://www.mathworks.com/matlabcentral/fileexchange/
2. fastPDBread - https://www.mathworks.com/matlabcentral/fileexchange/35009-a-fast-method-of-reading-data-from-pdb-files/all_files
3. bindata (P. Mineault; https://xcorr.net/?p=3326, http://www-pord.ucsd.edu/~matlab/bin.htm
4. parametrizeScFac.m from InSilicoTEM, as described in Vulovic M. et al., Image formation modeling in cryo-electron microscopy, J. Struct. Biol. (2013) 183(1):19-32.
5. Rotation matrix sets were generated using code referenced in: Generating Uniform Incremental Grids on SO(3) Using the Hopf Fibrations, International Journal of Robotics Research, IJRR 2009 Anna Yershova, Swati Jain, Steven M. LaValle, and Julie C. Mitchell.


