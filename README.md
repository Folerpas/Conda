# Test description
* Cebreros 2022
  - El incendio comenzó el 18/07/2022 a las 15:30 (13:30 h metetorológica), con 1 pto de ignición en
    - Lat: 40.459320°  Lon: -4.468689°
  - Total área quemada 4264 ha
  - La simulación se iniciará ∼ 12:30 h antes (spin up) el 18/07/2022 a la 1:00 y terminará el 19/07/2022 a las 23:00 (34h)
  - fire_ignition_start_time1 = 44280 (s), son 12:30 h después que empieza la simulación, fire_ignition_end_time1 = 87480 (s), son 24:30 h después que empieza la simulación (12 h) después que inicia el fuego.

## Goals

Run WRF-SFIRE with LES - HPC_Vega

## Details

* Number of Domains: 4 
* Resolution: 9 km, 3 km, 1km, 200m,
* parent_grid_ratio: 3,3,3,5
* Resolution SFIRE: 10 m (a razón de 20)
* sr_x, sr_y: 20
* nodos(max # of recommended nodes): d01_8, d02_9, d03_15, d04_55

* namelist.input:
  - Without tracer_opt variable
  - feedback = 1,

* Parametrizaciones physics and dynamics
  <pre>
  &physics                 
  mp_physics               = 8,        8,        8,        8,
  ra_lw_physics            = 4,        4,        4,        4,
  ra_sw_physics            = 1,        1,        1,        1,
  radt                     = 15,       15,       15,       15,
  sf_sfclay_physics        = 1,        1,        1,        1,
  sf_surface_physics       = 2,        2,        2,        2,
  bl_pbl_physics           = 1,        1,        1,        0,
  bldt                     = 0,        0,        0,        0,
  cu_physics               = 3,        3,        0,        0,
  cudt                     = 0,        0,        0,        0,
  isfflx                   = 1,

  &dynamics                
  w_damping                = 1, 
  diff_opt                 = 2,
  km_opt                   = 4,        4,        4,         2,
  sfs_opt                  = 0,        0,        0,         2,
  epssm                    = 0.3,      0.9,      0.9,       0.9,
  </pre>

  - Los cambios de la physics solo hacerlos después de correr el ndown


## Results

### WPS 

* serial mode
  - COMPLETED   07:13:41 

### WRF
#### real.exe 

* nodes: 12
  - COMPLETED   00:17:47

  <pre>
  --- WARNING: Reduce the MPI rank count, or redistribute the tasks.(d01-d02)
  d01 2022-07-19_23:00:00 real_em: SUCCESS COMPLETE REAL_EM INIT
  </pre>

#### wrf.exe (d01 and d02) 
* Run wrf.exe for the d01 and d02

  - max_dom = 2
  - nodes = 9 (with 8 nodes runs faster and without warnings)
    - time_step = 56

        <pre>
        The time step is probably too large for this grid distance, reduce it.
        If you are sure of your settings, set reasonable_time_step_ratio in namelist.input >    6.42071199
        --- FATAL CALLED ---------------
        FATAL CALLED FROM FILE:  <stdin>  LINE:     384
        --- ERROR: Time step too large
        -------------------------------------------
        </pre>

    - time_step = 54
      - COMPLETED   01:32:20

        <pre>
        --- WARNING: Reduce the MPI rank count, or redistribute the tasks. (d01)
        d01 2022-07-19_23:00:18 wrf: SUCCESS COMPLETE WRF
        </pre>
  
#### ndown (d03) 
* Delete or save to another directory wrfout_d01 outputs
* Rename wrfout_d02* to wrfout_d01*

* Rename wrfinput_d03 to wrfndi_d02 (this is the name the program expects)

* Modify namelist.input
    - Make sure io_form_auxinput2 = 2 is set in the &time_control section of the namelist.
      <pre>
      &domains                 
      time_step                = 18,
      max_dom                  = 2,
      e_we                     = 511,      679,     1141,
      e_sn                     = 370,      583,     1096,
      dx                       = 3000,     1000,      200,
      dy                       = 3000,     1000,      200,
      i_parent_start           = 1,        142,      229,
      j_parent_start           = 1,        91,      185,

      &physics
      cu_physics               = 3,        0,        0,

      &dynamics
      w_damping                = 1, 
      epssm                    = 0.9,      0.9,       0.9,
      </pre>

* Run ndown.exe
  - This produces a wrfinput_d02 and wrfbdy_d02 file (both which will actually correspond to domain 03).
  - 9 nodes
    - COMPLETED   00:09:28
      <pre>
      ndown_em: SUCCESS COMPLETE NDOWN_EM INIT
      </pre>

#### wrf.exe (d03) 
* Rename wrfinput_d02 to wrfinput_d01.
* Rename wrfbdy_d02 to wrfbdy_d01.

* Rename (or move) the wrfout_d01* files to something else (or another directory)
* Modify namelist.input
    <pre>
      &domains                 
      time_step                = 6,
      max_dom                  = 1,
      e_we                     = 679,     1141,
      e_sn                     = 583,     1096,
      dx                       = 1000,      200,
      dy                       = 1000,      200,

      &physics
      cu_physics               = 0,        0,
    </pre>

* Run wrf.exe 
  - 18 nodes
    - COMPLETED   02:07:00
        <pre>
          d01 2022-07-19_23:00:00 wrf: SUCCESS COMPLETE WRF
        </pre>

#### ndown.exe (d04)  
* Rename wrfinput_d04 to wrfndi_d02 

* Run ndown.exe for d04 
    <pre>
      &domains                 
      time_step                = 6,
      max_dom                  = 2,
      e_we                     = 679,     1141,
      e_sn                     = 583,     1096,
      dx                       = 1000,    200,
      dy                       = 1000,    200,
      i_parent_start           = 1,       229,
      j_parent_start           = 1,       185,
      parent_grid_ratio        = 1,       5,
      parent_time_step_ratio   = 1,       5,
      sr_x                     = 0,       20,
      sr_y                     = 0,       20,

      &dynamics
      km_opt                   = 4,         2,
      sfs_opt                  = 0,         2,

      &fire
    </pre>

* run ndown.exe
  - 18 nodes
    - COMPLETED   00:28:41
      <pre>
      ndown_em: SUCCESS COMPLETE NDOWN_EM INIT
      </pre>

#### wrf.exe (d04) *
* Rename wrfinput_d02 to wrfinput_d01.
* Rename wrfbdy_d02 to wrfbdy_d01.

* Rename (or move) the wrfout_d01* files to something else (or another directory)
* Modify namelist.input (d04)
    <pre>
        &domains                 
        time_step                = 1.2,
        max_dom                  = 1,
        e_we                     = 1141,
        e_sn                     = 1096,
        dx                       = 200,
        dy                       = 200,
        sr_x                     = 20,
        sr_y                     = 20,

        &physics                 
        bl_pbl_physics           = 0,
        cu_physics               = 0,

        &dynamics                
        km_opt                   = 2,
        sfs_opt                  = 2,

        &fire
    </pre>

* 55 nodes
    - FAILED   00:00:31
      <pre>
      taskid: 0 hostname: cn0133
       module_io_quilt_old.F        2931 F
      -------------- FATAL CALLED ---------------
      FATAL CALLED FROM FILE:  <stdin>  LINE:    6910
      ERROR reading namelist domains
      </pre>

    - Solution
      <pre>
        time_step                = 1,
        time_step_fract_num      = 2,
        time_step_fract_den      = 10,
        feedback = 0
      </pre>

      - time_step is an integer variable. If you want to use 1.2 sec as your time_step, set time_step = 1, time_step_fract_num = 2 and time_step_fract_den = 10.

      - Paró después de generar el primer fichero (en este plazo de tiempo 2022-07-18_01:03:45). Signal 11 en el wrf.log

* 45 nodes
* time_step = 0.8
    - Paró después de generar el primer fichero (en este plazo de tiempo 2022-07-18_01:03:45). Signal 11 en el wrf.log

* 50 nodes
* #SBATCT --mem=12800GB    
* time_step = 1.2

    rsl.error.0000
    <pre>   
      [cn0001:3639441:0:3639441] ib_mlx5_log.c:171  Transport retry count exceeded on mlx5_0:1/IB (synd 0x15 vend 0x81 hw_syn d 0/0)
    </pre>

    wrf.log
    <pre>
      mpirun noticed that process rank 0 with PID 0 on node cn0001 exited on signal 6 (Aborted).
    </pre>

