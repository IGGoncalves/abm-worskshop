# XI Spanish Chapter Meeting (ESB): Hands-on PhysiCell Workshop

[TOC]

## Running PhysiCell online

PhysiCell [1] works on **Windows, Mac and Linux**. However, it is written in **C++**, which means that it requires a compiler. If you are on Windows, for example, you will need to prepare your development environment before being able to run any PhysiCell examples. Furthermore, you will most likely need some **additional tools to analyze your simulation results** (e.g., Python, ImageMagick, MATLAB, ParaView, ...).

To avoid having to install all these dependencies and potentially run into combability issues during the workshop, **we prepared a ready-to-run virtual container** that runs as a Linux machine with Python already installed. The PhysiCell code and Python scripts used in this course are also already available in the container.

The container is hosted on Gitpod, and you should click on the button below to access it:

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/IGGoncalves/abm-worskshop)

Alternatively, follow this link: https://gitpod.io/#https://github.com/IGGoncalves/abm-worskshop

> ⚠️ Note: Gitpod offers a **free tier** with a total of **50 computation hours per month**, which is more than enough for the workshop. You are invited to test some of the examples described in this documentation guide, but be aware of this limit.

### Creating a Gitpod account

Once you click on the button above, you will be prompted to login to Gitpod. If you already have an account on GitHub you will be able to login using your account.

If you don't have a GitHub account, please **create one**. You will need to **grant permission to Gitpod to read your email address**, and Gitpod may require that you further validate your account with **a code that will be sent to a mobile phone number you provide** if your GitHub account is very recent. ![steps](D:\Chapter\steps.png)

### The Gitpod environment

When asked what code editor you would like to use, please select **VSCode**.  Next, you should be able to access the container, which looks like this:

![base](D:\Chapter\base.png)

Everything is all setup to go through this workshop 🥳

-----

## Running your first model

PhysiCell comes with some **template models** that run out-of-the-box. We will start by working with the `cancer-biorobots` example but the rules used to run this simulation apply to the other template models. For more information on this model, you can check the [full documented version](https://nanohub.org/resources/pc4cancerbots) [2].

First, we need to **populate our workspace** with the files for the project that we chose. In our case, this can be done by running the following command in the terminal:

```bash
make cancer-biorobots-sample
```

> Note: The cancer-immune-sample project was slightly modified to better fit this course. All models rules are the same, but the simulation times were decreased. Therefore, the configuration file may not match the one offered by PhysiCell.

Then, we need to compile the project:

```bash
make
```

which will result in an executable that can be run to simulate our model. The name of this executable is `cancer-immune` and it can be run by calling the command:

```bash
./cancer-immune
```

### Qualitative analysis

After running your simulation, you will obtain some `SVG` files that can be found in the `output` folder. You can **manually select some of these files and inspect them** but it's also possible to run the following command and **obtain a `GIF` showing the model's evolution over time**. This file will be saved to the `output` folder as well.

```bash
make gif
```

### Quantitative analysis

In order to perform some **more advanced data analysis routines**, such as plotting the concentration of a given substance in the microenvironment, or getting the cells' trajectories and numbers over time, we will be using Python. [`physicool`](https://physicool.readthedocs.io/en/latest/?badge=latest) [3] is a library tailored to PhysiCell projects that allows you to run parameter estimation and  optimization studies. In addition, it offers some data analysis features, which we will be using. First of all, you should install it in your environment with `pip`.

```bash
pip install physicool
```

Now, you can follow the `visualize_ouputs.ipynb` notebook which includes some Python code to inspect the simulation results.

### Testing other projects

Feel free to test some of the other PhysiCell template projects and look at the results! 

You will first need to reset your workspace to its original status (cleaning output and compiled files,...) by running these commands.

```bash
make data-cleanup
make clean
make reset
```

You can check the names of the available projects by running the following command inside the terminal:

```bash
make list-projects
```

For this workshop, we will not get into the intracellular models. Thus, you will be able to choose between:

```
Sample projects: template biorobots-sample cancer-biorobots-sample cancer-immune-sample celltypes3-sample heterogeneity-sample pred-prey-farmer virus-macrophage-sample worm-sample interaction-sample
```

Once you have selected a project, just run the command `make <project_name>` where `<project-name>` should be replaced by the name you chose. Compile your code using `make`. The name of the executable file will be displayed once the code has stopped compiling. In case you missed it, run `ls` inside the terminal to list the files inside the current directory. The executable file will be displayed in green, making it easily identifiable.

## Working with models

### Preparing the model

In this section, we will be working with one of the simplest template models: `template`. Let's prepare our workspace by running the following commands. Let's also run a simulation with the original code and save the results (the `GIF` file) to our local machines.

```bash
make data-cleanup
make clean
make reset
make template
make
./project
make gif
```

### Changing model parameters

The simplest way  to change PhysiCell models is through the `PhysiCell_settings.xml` file, which can be found in the `config` folder. Here, you can find the definition for the substances and cells present in the simulation. You can also modify the size of the simulation domain and the simulation time length.

Let's try to change the cell proliferation rate and the number of initial cells in the simulation. Let's change the durations to [300, 300, 180, 60]:

```xml
<phase_durations units="min"> 
    <duration index="0" fixed_duration="false">300.0</duration>
    <duration index="1" fixed_duration="true">300</duration>
    <duration index="2" fixed_duration="true">180</duration>
    <duration index="3" fixed_duration="true">60</duration>
</phase_durations>
```

and the number of cells to 10:

```xml
<user_parameters>
    <random_seed type="int" units="dimensionless">0</random_seed> 
    <!-- example parameters from the template --> 
    <div_initialization type="divider" description="---Initialization settings---"/>
    <number_of_cells type="int" units="none" description="initial number of cells (for each cell type)">10</number_of_cells>
</user_parameters>
```

When we do changes to the config file only, we do not need to recompile the code again. Let's just clean the old data with `make data-cleanup` and run the project again with `./project`.

Feel free to try changing other model parameters and see how it impacts the results! 😼 For example, you could try other proliferation rates, changing the apoptosis and necrosis rates, make the cells move at different speeds (make sure to enable motility in the configuration file) or change the random seed.

### Adding model extensions

When we want to **introduce changes to how the cells behave**, the XML file might not be enough, and we will need to modify the `custom.cpp` file (found in the `custom` folder) to reflect the new rules we want to add. As an example, we will be working with a [**model extension**](https://github.com/m2be-igg/PhysiCell-ECM) that introduces the **extracellular matrix (ECM)** as a non-diffusing substance and that takes into account the ECM concentration to **regulate how cells move** [4].

First, let's add the ECM as a substance in the `PhysiCell_settings.xml` file:

```xml
<variable name="ECM" units="mg/mL" ID="1">
    <physical_parameter_set>
        <diffusion_coefficient units="micron^2/min">
            0.0
        </diffusion_coefficient>
        <decay_rate units="1/min">
            0.0
        </decay_rate>
    </physical_parameter_set>
    <initial_condition units="mg/mL">
        2.5
    </initial_condition>
    <Dirichlet_boundary_condition units="mg/mL" enabled="false">
        2.5
    </Dirichlet_boundary_condition>
</variable>
```

Given that cells in the `template` project do not move, let's change this by enabling cell motility. We will also be setting the `migration_bias` to 0, to make cells move according to a random walk.

```xml
<motility>  
    <speed units="micron/min">1</speed>
    <persistence_time units="min">1</persistence_time>
    <migration_bias units="dimensionless">0</migration_bias>

    <options>
        <enabled>true</enabled>
        <use_2D>true</use_2D>
    <chemotaxis>
        <enabled>false</enabled>
        <substrate>substrate</substrate>
        <direction>1</direction>
    </chemotaxis>
    </options>
</motility>
```

Next, let's go to the `custom.cpp` file and change the way cells move. We will introduce a new function that samples the concentration at the cell's current voxel, estimates the dynamic viscosity of that voxel based on empirical measurements, and uses this value to calculate the cell's equation of motion:

```c++
void drag_update_cell_velocity( Cell* pCell, Phenotype& phenotype, double dt ) {
    // sample ECM
    int ECM_density_index = microenvironment.find_density_index( "ECM" );
    double ECM_density = pCell->nearest_density_vector()[ECM_density_index];
    double dyn_viscosity;
    
    // get viscosity based on concentration
    if(ECM_density == 2.5) {
        dyn_viscosity = 7.96;
    }
    else if(ECM_density == 4.0){
        dyn_viscosity = 18.42;
    }
    else if(ECM_density == 6.0) {
        dyn_viscosity = 39.15;
    }
    
    // update velocity
    standard_update_cell_velocity(pCell, phenotype, dt);
    // include the 1/vu (1/ECM density) term to consider friction
    pCell->velocity /= dyn_viscosity;
    return;
}
```

We also need to go to the cell definition function and define that we will be using this new function to calculate the cell velocities instead of the standard `standard_update_cell_vecolcity`:

```c++
cell_defaults.functions.update_velocity = drag_update_cell_velocity;
```

Lastly, we need to define this new function in the `custom.h` file:

```cpp
void drag_update_cell_velocity( Cell* pCell, Phenotype& phenotype, double dt );
```

-------

## Running parameter estimation studies

In this section, we will be exploring a **chemotaxis** model. In PhysiCell, chemotaxis is defined by the `migration_bias` parameter, which dictates **how sensitive** cells are to a given substance. If this value is set to 0, the cells will move according to a **random walk**. Conversely, if the value is set to 1, they will deterministically **follow the substance gradient**.

We will be working with a model where there will be an **oxygen source on one of the domain walls**, and **a group of cells on the opposite wall**. On the one hand, for the cell population with **random motility,** we expect that they will remain **close to their initial position** as they will move around without following any specific direction.  On the other hand, **cells that follow oxygen will move towards the opposite wall**.

Our goal is to understand if, given some generated data of the final cells' position, we can predict the **migration_bias** and **speed** values that  originated this dataset.

### Resetting the environment

Just to be sure, let's get our environment to its initial state.

```bash
make data-cleanup
make clean
make reset
```

### Preparing the model

#### Running the model as a template

```bash
make abm-template
```

#### Building the model from scratch

We will be using the `template` project once again, so we can populate our environment with these files:

```bash
make template
```

However, we are going to **introduce some model changes**. Specifically, we will be **adding an oxygen source** on one of the domain walls and make our **cells move towards oxygen gradients**.

We need to change the ``PhysiCell_settings.xml` file that can be found in the `config` folder. Firstly, let's **decrease the total simulation time** since this is a short experiment.

```xml
<max_time units="min">360</max_time> 
```

Subsequently, in the same file, let's **add oxygen as a substance** below the definition for the "substrate" substance.

```xml
<variable name="oxygen" units="dimensionless" ID="1">
	<physical_parameter_set>
		<diffusion_coefficient units="micron^2/min">100000.0</diffusion_coefficient>
		<decay_rate units="1/min">10</decay_rate>
	</physical_parameter_set>
	<initial_condition units="mmHg">0</initial_condition>
	<Dirichlet_boundary_condition units="mmHg" enabled="true">0</Dirichlet_boundary_condition>
	<!--  use this block to set Dirichlet boundary conditions on individual boundaries  -->
    <Dirichlet_options>
        <boundary_value ID="xmin" enabled="false">0</boundary_value>
        <boundary_value ID="xmax" enabled="false">0</boundary_value>
        <boundary_value ID="ymin" enabled="false">0</boundary_value>
        <boundary_value ID="ymax" enabled="true">100</boundary_value>
        <boundary_value ID="zmin" enabled="false">0</boundary_value>
        <boundary_value ID="zmax" enabled="false">0</boundary_value>
    </Dirichlet_options>
</variable>
```

Now, let's define that the **cells should follow oxygen gradients**. Enable cell motility, chemotaxis, and change "substrate" to "oxygen".

```xml
<motility>  
    <speed units="micron/min">1.0</speed>
    <persistence_time units="min">120.0</persistence_time>
    <migration_bias units="dimensionless">0.2</migration_bias>
    <options>
        <enabled>true</enabled>
        <use_2D>true</use_2D>
        <chemotaxis>
            <enabled>true</enabled>
            <substrate>oxygen</substrate>
            <direction>1.0</direction>
        </chemotaxis>
    </options>
</motility>
```

Finally, we need to define that we will be using an external file to set the **initial position of the cells**:

```xml
<initial_conditions>
    <cell_positions type="csv" enabled="true">
        <folder>./config</folder>
        <filename>cells.csv</filename>
    </cell_positions>
</initial_conditions>
```

and we will update the `cells.csv` file to **place the cells close to the ymin wall**.

```
-450,-450,0,0
-350,-450,0,0
-250,-450,0,0
-150,-450,0,0
-5,-450,0,0
5,-450,0,0
150,-450,0,0
250,-450,0,0
350,-450,0,0
450,-450,0,0
```

Since we will be using a CSV file to get the initial cell positions, we no longer need these lines in the `setup_tissue` function found in the `custom.cpp` file and they can be erased:

```cpp
double Xmin = microenvironment.mesh.bounding_box[0]; 
double Ymin = microenvironment.mesh.bounding_box[1]; 
double Zmin = microenvironment.mesh.bounding_box[2]; 

double Xmax = microenvironment.mesh.bounding_box[3]; 
double Ymax = microenvironment.mesh.bounding_box[4]; 
double Zmax = microenvironment.mesh.bounding_box[5]; 

if( default_microenvironment_options.simulate_2D == true )
{
    Zmin = 0.0; 
    Zmax = 0.0; 
}

double Xrange = Xmax - Xmin; 
double Yrange = Ymax - Ymin; 
double Zrange = Zmax - Zmin; 

// create some of each type of cell 

Cell* pC;

for( int k=0; k < cell_definitions_by_index.size() ; k++ )
{
    Cell_Definition* pCD = cell_definitions_by_index[k]; 
    std::cout << "Placing cells of type " << pCD->name << " ... " << std::endl; 
    for( int n = 0 ; n < parameters.ints("number_of_cells") ; n++ )
    {
        std::vector<double> position = {0,0,0}; 
        position[0] = Xmin + UniformRandom()*Xrange; 
        position[1] = Ymin + UniformRandom()*Yrange; 
        position[2] = Zmin + UniformRandom()*Zrange; 

        pC = create_cell( *pCD ); 
        pC->assign_position( position );
    }
}
std::cout << std::endl; 
```

The model is now finished and we can move on to the `black_box.ipynb` notebook to run some simulations.

### Running parameter estimation studies

The `black_box.ipynb` and `optimizer.ipynb` notebooks will guide you through running PhysiCell models with `physicool` and conducting parameter exploration and optimization studies.

## References

[1] Ghaffarizadeh, A., Heiland, R., Friedman, S. H., Mumenthaler, S. M.,  & Macklin, P. (2018). PhysiCell: An open source physics-based cell  simulator for 3-D multicellular systems. In T. Poisot (Ed.), PLOS  Computational Biology (Vol. 14, Issue 2, p. e1005991). Public Library of Science (PLoS). https://doi.org/10.1371/journal.pcbi.1005991

[2] Randy Heiland, Paul Macklin (2020), "PhysiCell cancer biorobots simulation," https://nanohub.org/resources/pc4cancerbots. (DOI: 10.21981/050C-PM63).

[3] Gonçalves, Inês G, Hormuth, David, Phillips,  Caleb, & Prabhakaran, Sandhya. (2022). PhysiCOOL (v0.2.1). Zenodo.  https://doi.org/[10.5281/zenodo.6458585](https://doi.org/10.5281/zenodo.6458585).

[4] Gonçalves, I. G., & Garcia-Aznar, J. M. (2021). Extracellular matrix density regulates the formation of tumour spheroids through cell  migration. In P. K. Maini (Ed.), PLOS Computational Biology (Vol. 17,  Issue 2, p. e1008764). Public Library of Science (PLoS).  https://doi.org/10.1371/journal.pcbi.1008764