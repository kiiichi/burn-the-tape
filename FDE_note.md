# FDE_Learning

The note of FDE learning 
***
Based on [Ansys](https://courses.ansys.com/index.php/learning-track/ansys-lumerical-fde/ "Don't touch me !")
***

## 1. Characteristic

1. FDE solver can be used to characterize the supported modes of light-guiding structures, to obtain results such as the mode profile, the effective index, loss, dispersion, group velocity. 

2. FDE solver can be used to optimize the power coupling between modes. 

3. FDE solver can deal with structures where the cross section is uniform ( straight or bent ) , or helical with a uniform period along the direction of propagation. 

***

## 2. Workflow

 1. Set up

    - Add/dfeine materials
    - Add structure
    - Add simulation region and mesh

 2. Run

    - Switch to analysis mode
    - Opens Eigensolver Analysis window

 3. Fide modes

    - Set the mode analysis parameters
    - Mesh the structure
    - Calculates the modes

 4. Analysis

    - Model analysis
    - Frequency sweep
    - Overlap analysis

 5. Switch to layout

 6. Modify simulation

You can iterate the step **'Run'** to **'Modify simulation'**

The **built-in parameter sweep** and **optimization tool** can be used to help automate this process
***

## 3. Setting Up the Simulation

Demo for Soi rib waveguide

### Steps

1. Add a rectangle as the glass substrate

   click **Structure** -> **rectangle**

   - In the **Geometry** tab , both set the **x span** & the **y span** is 10 μm

   - In the **Material** tab , set the **material** is **SiO2(Glass) - Palik**

   - Rename the object **SiO2**

2. Add another rectangle as the silicon waveguide

   click **Structure** -> **rectangle**

   - In the **Geometry** tab , set the **x span** is 0.5 μm and set the **y span** is 0.22 μm

   - In the **Material** tab , set the **material** is **Si (silicon) - Palik**

   - Rename the object **Si**

3. Add the engienmode solver after the structure is defined

   click **Simulation** -> **Eigenmode solver**

   - In the **General** tab , set the **background index** is 1
   - In the **Geometry** tab , both set the **x span** & the **y span** is 4 μm
   - In the **Boundary condition** tab set the bc is **metal**
   - In the **Material** tab , select the **fit materials with multi-coefficient moder** and the **fit sampled materials** option , and then set the **wavelength** range is 1.5 μm to 1.6 μm

4. Add a mesh override region

   click **Simulation** -> **Mesh**

   - In the **General** tab , select the **set maxmium mesh step** option , and both set the **dx** & **dy** is 0.02 μm
   - In the **Geometry** tab , select the **based on a structure** option and type 'Si' in the **structure**

***

## 4. Run Demo

### Steps

1. Click **Run** , in the **Engiensolver Analysis** window , we can set the calculation parameters

   - In the **Modal analysis** tab , set the **wavelength** is 1.5 μm and the **number of trial modes** is 10
   - Assure the **use max index** was be selected
   - Click the **mesh structure** to display the material properities in the plot area
   - Click the **calculate modes** to start this calculating

   - When the calculating is finished , we can select a mode what we want in the **Mode list** and see its field profile in the plot area

2. Recalculate the modes using a finer mesh

   - Click **Layout**
   - Edit the mesh override object and both set the mesh is 0.01 μm in **dx** and **dy**

We can iterate these steps to get optimal results