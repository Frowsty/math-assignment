
# Homeautomationsystem

I want to create a complete Homeautomationsystem

The system should contain door sensor in form of magnetic contacts, a camera system that tracks the person.
Light automation, turn on the lights depending on how dark it it.
Control the colors of the lights by inputting a HEX value and have the system convert it to a readable RGB value for the lights.

## Hardware-Requirements

|  Type          | Description |
|  ------------  | :------------: |
| Temperature Sensor  | Reads temperature|
| Light Sensor        | Reads light levels|
| LIDAR Sensor	      | Reads movement|
| Magnet contacts     | For doors (alarm system) |
| Smart lights ( RGB )| Lights up the room in beautiful colors|


## Problems to solve

- Solving the singal sent from door magnet contacts to the main system
    - How to send the signal from the contacts to the system
- Read the light levels in the room
    - Allow user to input HEX value for RGB function of the lights
    - Convert HEX to RGB (decimal values)
- LIDAR sensor should track distance between sensor and objects around the main door when alarm system is on
    - Track distance, any deviation from normal will trigger the alarm
- Temperature sensor
    - Tracks the temperature in the building and adjust heating accordingly

Determine what equations should be used where and what formula is required to solve the problems.

## Solving the magnet contact signal
We can solve this through code by assuming we're sending a couple of bits (0 or 1) from the magnet contacts
This is pure theoretical code, it can run but it serves no actual purpose. The important bit here is simply the AND / boolean comparison
```cpp
#include <iostream>

bool signal = true; // default the value to 1 (magnetic contacts indicate door is closed if 1 is passed)

// placeholder for retrieve_signal function, this in theory should send actual sensor data. 
bool retrieve_signal(int m)
{
    if (m)
        return true
    else
        return true
}

// listener
function start_listener(bool& s)
{
    while (s)
    {
        // Continuously retrieve the signal from the magnets 
        s = retrieve_signal(0) && retrieve_signal(1); // retrieve the signal from both of the magnets
    }
}

bool alarm_on(bool v)
{
    return v;
}

int main()
{

    // only check if the alarm system is on
    if (alarm_on(true))
    {
        // infinite loop that continuously checks the signal from the magnets
        start_listener(&signal);

        // if we ever reach this point the signal has been set to 0
        // which would indicate the door has opened while the alarm is turned on
        std::cout << "ALARM SOUNDS IN 10 SECONDS\n";
    }

    return 0;
}
```

In this problem we use simple boolean algebra in terms of an AND gate. The magnet contact that is in direct connection with the system will
always send the signal 1 where as the other magnet only sends 1 if it is making contact with the main magnet. This gives us the ability to used
the boolean operation AND or in this case '&&' to make sure we set the signal value to 1 if magnet 0 AND magnet 1 equals to 1. As soon as magnet 1 sends a 0 the signal is broken and our AND operation will return 0 which will sound the alarm system

## Solving the temperature sensor signal
The temperature sensor sends the Temperature
Every minute the sensor gathers temperature. For 2 hours we gather temperature only. Every 2 hours we calculate the average temperature and the average deviance.

The equation would look something like this
t = Time = 120 minutes
X = Value = Temperature every minute
A = Average = (X1 + X2 + Xt) / t
Deviation = sqrt(sum of(X - A)^2/t)

Using the following equation we can calculate the population standard deviation
![equation](https://www.bizskinny.com/images/population-standard-deviation-formula.PNG)
## Solving the hex conversion to decimal (RGB)

The conversion from HEX to RGB is very simple, to convert a hex value to RGB simply take the HEX color code and split it into 3 pieces.
Example #32A852 -> R: #32, G: #A8, B: #52

Once we have split it into 3 pieces we can calculate each piece separately

R: #32 -> (3 * 16) + 2 = 50
G: #A8 -> (10 * 16) + 8 = 168
B: #52 -> (5 * 16) + 2 = 82

In code we don't have to convert hex to decimal but rather just split the hex value into 3 pieces.
The computer automatically converts it for us, an int can store hex, bytes and decimal values.

The solution would look something like this in code
```cpp
struct RGB
{
	int r;
	int g;
	int b;
};

RGB hex_to_rgb(int hex)
{
	RGB rgb_color;
	rgb_color.r = ((hex >> 16) & 0xFF);
	rgb_color.g = ((hex >> 8) & 0xFF);
	rgb_color.b = ((hex) & 0xFF);
	return rgb_color;
}

int main()
{
	RGB test_color;
	int test = #32A852;
	test_color = hex_to_rgb(test);
	
	std::cout << "R: " << test_color.r << " G: " << test_color.g << " B: " << test_color.b << std::endl;
	return 0;
}

// expected output R: 50 G: 168 B: 82
```
Some programs use 0 - 1 for RGB rather than 0 - 255 in which case we would just divide by 255 to get the 0 - 1 value
Example:
`rgb_color.r = ((hex >> 16) & 0xFF) / 255.f;`

## Solving the LIDAR sensor problems

Lidar stands for light detection and ranging, we can use this to measure distance between the lidar sensor and an object. In my use case we will point the lidar sensor at the main door of the building to track distance between the sensor and anything walking into the door.
Lidar determines distance with the following equation

`d = c * t / 2`
d = distance
c = the constant of speed of light
t = time between lidar giving out a light signal to when it is received again

In a program we can continuously search for deviance between what values we expect from the sensor vs what we don't expect when the alarm system is on.

In code we can write it as following

```cpp
#include <iostream>
#include <cstdlib>
#include <time.h>

using namespace std;

const uint32_t LIGHT_SPD = 299792458; // speed of light
const float E_DIST = 10.f; // expected distance in meters
const float P_ERROR = 0.5f; // possible distance error
const bool ALARM_ON = true;

double time_to_distance(double t)
{
	return (LIGHT_SPD * t) / 2;
}

double get_lidar_data()
{
    double random_number = rand() % 10;
    
    return random_number * 0.00000001;
}

int main()
{
    
    srand(time(NULL));
    
    // make sure the alarm is on
    if (ALARM_ON)
    {
        double distance = time_to_distance(get_lidar_data());
        // continuously check for any deviation
        while (distance >= (E_DIST + P_ERROR) || distance >= (E_DIST - P_ERROR))
        {
            std::cout << "Distance: " << distance << "m\n";
            distance = time_to_distance(get_lidar_data());
        }
        
        // deviation past P_ERROR happened
        std::cout << "SOUND ALARM, distance: " << distance << "m is less than allowed" << std::endl;
    }
	return 0;
}
```
