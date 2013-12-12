# TRICAL

TRICAL is a UKF-based iterative scale and bias calibration algorithm for
tri-axial field sensors (e.g. magnetometers).

The implementation is based on the unscented filter formulation described in
[Real-Time Attitude-Independent Three-Axis Magnetometer Calibration][1];
performance is similar to TWOSTEP but it's less computationally intensive, and
able to provide iterative calibration estimates.

[1]: http://www.acsu.buffalo.edu/~johnc/mag_cal05.pdf


## Overview

TRICAL is configured with an expected field norm (defaulting to 1.0). In the
case of a magnetometer, this would be the magnitude of *B* at its current
location (as output by the WMM, for example).

The input to the calibration process is a sequence of 3-vectors representing
the field readings from the sensor. These are in the same units as the field
norm.

The calibration parameters used are a 3-vector representing the estimated
bias, and a 3x3 symmetric matrix representing the estimated scale error.

These parameters allow a magnetometer to be corrected for hard and soft iron
distortion, sensor bias, sensor scale error, and sensor non-orthogonality. An
accelerometer can be corrected for bias, scale error and non-orthogonality.

## Usage

First, `#include "TRICAL.h"`, and declare a `TRICAL_instance_t` in whatever
scope is appropriate. Call `TRICAL_init(…)` with your TRICAL instance as a
parameter, then (if desired) set the expected field norm and measurement
noise.

The field norm should be the magnitude of the calibrated readings; TRICAL will
scale your measurements to reach that value. If possible, it should be fairly
close to the magnitude of the measurements themselves, since that will reduce
the time taken to converge on an estimate of the calibration parameters.

The measurement noise should be something like the standard deviation of your
measurement error. If in doubt, you can use the RMS noise value from your
datasheet; it's not terribly critical to get an exact value for this, but it
does help the calibration estimate to converge at an appropriate rate.

Once you've initialized the instance, you can start giving it sensor readings
to use in estimating the calibration parameters. These readings are passed in
via `TRICAL_estimate_update(…)`; each update results in a new calibration
estimate, which you can access using `TRICAL_estimate_get(…)`.

To apply the current calibration estimate to a measurement, just call
`TRICAL_measurement_calibrate(…)`.

    #include "TRICAL.h"

    TRICAL_instance_t global_instance;

    /* ... */

    void your_init_proc(void) {
        /* ... */

        TRICAL_init(&global_instance);
        TRICAL_norm_set(&global_instance, 60.0);
        TRICAL_noise_set(&global_instance, 1.5);
    }

    void your_sensor_read_proc(void) {
        float sensor_reading[3];

        /* ... */

        TRICAL_estimate_update(&global_instance, sensor_reading);

        /* ... */

        float calibrated_reading[3];
        TRICAL_measurement_calibrate(&global_instance, sensor_reading,
                                     calibrated_reading);

        /* Now use calibrated_reading as an input to your AHRS or whatever */
    }


## Build instructions

Requires `cmake` version 2.8.7 or higher.

Create a build directory outside the source tree, then use cmake to generate
the makefile.

```
mkdir TRICAL_build
cd TRICAL_build
cmake /path/to/TRICAL
```

Now, build the library using the `make` command.


## Testing

The `googletest` library is used for unit testing. To build the unit tests,
use `make unittest`. The unit tests can then be executed by running
`test/unittest` in the build directory.


## Python module installation

Requires `cmake` version 2.8.7 or higher.

Run `python setup.py install` to build the C shared library and install the
Python interface (the `TRICAL` module) in your `site-packages` directory.

Alternatively, just run `pip install https://github.com/sfwa/TRICAL/archive/master.zip#egg=TRICAL-1.0.0`
to download and install.


## Compiling with Texas Instrumets Code Composer Studio 5

Import the root directory of this project (`TRICAL`) into your workspace. CCS
should search all contained files, and find the project files. Complete the
import, and build.
