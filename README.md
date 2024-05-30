# Quantification of cooperative stability by model fitting to FACS data 
Authors: Kuan-Lun Hsu and Chen-Hsiang Yeang
Last updated date: 2024/5/30 
Cite this work:
Hsu, KL. & Yeang, CH. QCS: Quantification of cooperative stablization by model fitting to GPS data v1.0.0 [Computer software] Zenodo https://zenodo.org/doi/10.5281/zenodo.11395486

## Description

This project aims to quantify the cooperative stability (increasing stability when protein is bound to its binding partner) by model fitting to experimental data. To this end, our group designed an experiment to measure the stability of RAD1 protein with different expression levels of its binding partner, HUS1 protein, and vice versa. To quantify the cooperative stability from this data, we built an ordinary differential equations(ODEs) model to describe the dynamic of a heterodimer formation system. We obtained the parameters by model fitting to the data.

## Installation
We recommend using Python 3.8 and Jupyter Notebook to run the model fitting.  Please also install the libraries listed in the requirements.txt 

- Install Python and Jupyter Notebook at Anaconda (https://www.anaconda.com/ )

- Using pip or conda to install/upgrade the required libraries

  ```bash
  $ python -m pip install --upgrade pip
  $ pip install -r requirements.txt
  ```
- Or download the following dependencies manually
 - numpy==1.24.3
 - scipy==1.10.0
 - sympy==1.12
 - matplotlib==3.5.3
 - seaborn==0.11.2
 - mpl_toolkits

For example:

 ```bash
  $ pip install numpy==1.24.3
 ``` 

## Implementation (DEMO)
- Once you have installed the Anaconda, you can launch the Jupyter Notebook via Anaconda-Navigator or Anaconda prompt:

  ```bash
  $ Jupyter Notebook
  ```

- Open the Main.ipynb and start to run the program.

# Outline of the notebook

## I. Mathematical Modeling
1. ODEs Model of the heterodimer formation system

  ```python
from sympy import *
p1 = Symbol('p1')
p2 = Symbol('p2')
p3 = Symbol('p3')
lam1 = Symbol('lam1')
lam2 = Symbol('lam2')
lam3 = Symbol('lam3')
K = Symbol('K')
c1 = Symbol('c1')
c2 = Symbol('c2')
eq1 = c1 - lam1*p1 - lam3*p3
eq2 = c2 - lam2*p2 - lam3*p3
eq3 = p3 - p1*p2*K

  ```
2. Solve the equations and obtain an analytical solution

  ```python
ans1 = solve(eq3, p3) 
ans2 = solve([p3-ans1[0], eq1],[p1, p3]) 
ans3 = solve([p3-ans1[0], eq2],[p2, p3]) 
ans4 = solve([p2-ans3[p2], p1-ans2[p1]],[p1, p2]) 
ans5 = solve([p3-ans3[p3], p1-ans4[1][0]],[p1, p3]) 
print('p1:\n', ans4[1][0], '\n')
print('p2:\n', ans4[1][1], '\n')
print('p3:\n', ans5[0][1], '\n')
print('p1+p3:\n', simplify(ans4[1][0]+ans5[0][1]))
  ```
   
## II. Data processing
1. Import Data
	
	The test data set is generated by random sampling 10000 data points from the original RAD1 and HUS1 FACS readout, containing gfp, rfp, and bfp signal per data point. 

  ```python
RAD1_plus_HUS1 = FACS_data('data/GPS_RAD1_HUS1_for_testing.txt')
HUS1_plus_RAD1 = FACS_data('data/GPS_HUS1_RAD1_for_testing.txt')
  ```
2. Set a cutoff to filter out data with saturated signal 
3. Exploring the Data by visualization 
4. Identify the linear range and eliminate data outside the range

## *. How to use your data
Please put your FACS data in the following format:

 - line1: name of this experiment or other description
 - line2: *gfp_value*\t*rfp_value*\t*bfp_value*
 - line3: *gfp_value*\t*rfp_value*\t*bfp_value*
 - ....
And save it as a .txt file in data folder, then modify the filename in the jupyternote book

  ```python
protein1_plus_protein2 = FACS_data('data/your_data1.txt')
protein2_plus_protein1 = FACS_data('data/your_data2.txt')
  ```

## III. Model Fitting
1. Compare the difference between RFP and BFP
2. Estimate the cooperative stability by model fitting

This function would perform model fitting to five parameters: p1 degrade rate constant, p2 degrade rate constant, p3 degrade rate constant, association constant, and n-factor for correcting the different intensity between rfp and bfp. It would start the model fitting with all possible combinations of [0.1, 1, 10] as the initial values for the five undetermined parameters.
The model fitting by using the testing data set takes around 

  ```python
data = RAD1_plus_HUS1
n_all_fit_result = []
n_all = []
n_fit_result = model_fitting_for_n(data)
n_res = np.array([i[4] for i in n_fit_result])
std = n_res.std()
med = np.median(n_res)
n_all.append(med)
n_all_fit_result.append(n_fit_result)
print(med, std)
  ```
  
Finally, we use the median value of all results fitted with different initial values.

  ```python
print('The cooperative stability of RAD1: ',np.median(Coor_stab_RAD1))
print('The cooperative stability of HUS1: ',np.median(Coor_stab_HUS1))
  ```



