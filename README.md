# Materials : 

## MQ-135 and other gas sensors :
- [MQ-135 Gas Sensor Pinout, Features, Alternatives, Datasheet & Uses Guide.pdf](https://github.com/Air-Gas-Quality/Air-Gas-Quality/files/8318365/MQ-135.Gas.Sensor.Pinout.Features.Alternatives.Datasheet.Uses.Guide.pdf)
- https://www.electronicoscaldas.com/datasheet/MQ-135_Hanwei.pdf
- https://ram-e-shop.com/?s=MQ&product_cat=0&post_type=product

## Environmental Analysis : 
- https://www.airnow.gov/aqi-and-health/
- [aqi-technical-assistance-document-sept2018.pdf](https://github.com/Air-Gas-Quality/Air-Gas-Quality/files/8318367/aqi-technical-assistance-document-sept2018.pdf)
- https://www.pranaair.com/blog/what-is-air-quality-index-aqi-and-its-calculation/
- https://www.airnow.gov/aqi/aqi-calculator-concentration/

## MQ-135 Cracking Datasheet : 

1) Basic Functionality : 
---> Potentiometer (SnO2 -- Semiconductor). 
---> Gases ---> Voltage increases/Resistivity decreases.


2) What we want to find ? 
- We want to find the Rs (Sensitivity resistance) in the fresh clean air (calibration) in 24 hrs, then save the average value.
- Measure Rs in various gaseous states.
- Ro is the R-original which represents the calibrated sensor value when connected in parallel with RL = 20 kOhms and T = 20 deg C.
- Then, we want to find the ratio `Rs/Ro`; where `Rs : Sensitivity Resistance in various gaseous statuses`, `Ro = Original Resistance in fresh air`.
- Then, we want to plug the ratio into the graph from the datasheet to find the possible corresponding gas : 

![MQ-135](https://user-images.githubusercontent.com/60224159/159438596-55ff8d4f-c548-4b93-8070-c7d9e8996670.png)

### Notice : All gases can be detected when Rs/Ro > 1, but at very low concentrations which states that the device may misbehave at this point.???

- Example : 

### Recall; `Rs/Ro` = 0.9;

### then : 

The Air Quality state is `moderate` and most probable harmful gases that may lead to this resistivity are : </br>
      1) Acetona @ 50 ppm. </br>
      2) Toleuon @ 70 ppm. </br>
      3) Alcohol @ 100 ppm. </br>
      4) Ammonia @ 100-150 ppm. </br>
      5) Co2 @ 150-180 ppm. </br>
      

3) Circuit : 

![image](https://user-images.githubusercontent.com/60224159/159455758-84ae0871-1c89-4de2-8568-1a1afeb8a56c.png)

Connecting RL (load resistance) in parallel with the Rs to load the current onto the sensor part (increasing the senstivity of the sensor).

then, I(s) = I(total) - I(load) = I(cc) - V(s)/R(L).

then, R(s) = V(s) / I(s).

4) Suggested Code : 

```cxx
struct GasLevelRange {
   public:
      float startValue = 0.0f;
      float endValue = 0.0f;
};

struct GasLevels {
   public:
      GasLevelRange co2Levels;
      GasLevelRange coLevels;
      GasLevelRange alcoholLevels;
      GasLevelRange ammoniaLevels;
      GasLevelRange tolueneLevels;
      GasLevelRange acetoneLevels;
};

bool isCalibrating = true;
const Memory* memory = new Memory();
// max voltage 
static const float MAX_VOLTAGE = 5.0f;
static const float SOURCE_CURRENT = 0.3f; 
// 20 kOhm load resistance
static const float LOAD_RESISTANCE = 20000;

float& getResistivity(float& current, volatile uint10_t& potentiometerValue) {
      // Value         : Max
      // potentioValue : 1023 Dec
      // voltage       : 5V
      // then, voltage = (potentioValue / 1023f) * 5.0V.
      // convert to percentage 
      const float rationalValue = potentiometerValue / 1023f; 

      // convert to voltage 
      const float conductivity = rationalValue * MAX_VOLTAGE;  

      // get resistivity
      float resistivity = conductivity / current;   
      
      return resistivity;
}

float& getSensorCurrent(volatile uint10_t& potentiometerValue) {
    // calculate I(s)
    float loadCurrent = potentiometerValue / LOAD_RESISTANCE;
    float sensorCurrent = SOURCE_CURRENT - loadCurrent;
    return sensorCurrent;
}

void plugIntoGraph(float& graphRatio) {
     const GasLevels* gasLevels = new GasLevels();
     if (graphRatio < 1) {
           // lies between 0.9 and 1
           if (graphRatio >= 0.9) {
                  // possible different gases concentrations (ppm) 
                  // at current resistivity

                  // possible co2 levels at this resistivity
                  gasLevels->co2Levels.startValue = 110f;
                  gasLevels->co2Levels.endValue = 175f;

                  // possible NH4 levels
                  gasLevels->ammoniaLevels.startValue = 100f; 
                  gasLevels->ammoniaLevels.endValue = 140;

                  // possible alcohol levels
                  gasLevels->alcoholLevels.startValue = 80f; 
                  gasLevels->alcoholLevels.endValue = 110f;

                  //....etc....
                  
            // lies between 0.9 and 1
           } else if (graphRatio >= 0.8) {
                  // possible co2 levels at this resistivity
                  gasLevels->co2Levels.startValue = 160f;
                  gasLevels->co2Levels.endValue = 200f;

                  // possible NH4 levels
                  gasLevels->ammoniaLevels.startValue = 140f; 
                  gasLevels->ammoniaLevels.endValue = 180;

                  // possible alcohol levels
                  gasLevels->alcoholLevels.startValue = 110f; 
                  gasLevels->alcoholLevels.endValue = 160f;
                  
                  //....etc....
           }
     }

}

int main() {
    // analogValue = [0, 1023] (Decimal Encoded value) 
    // = [0V, 5.0V] (Real Voltage) 
    // = [0000000000, 1111111111] (in 10-bit binary expression).
    volatile uint10_t potentiometerValue = analogRead(A0);
    // sensor current
    float sensorCurrent = getSensorCurrent(potentiometerValue);
    // get the resistivity
    float resistivity = getResistivity(potentiometerValue, sensorCurrent);
    // save the calibration value)
    if (isCalibrating && !(memory->hasData())) {
      memory->save(resistivity);
      isCalibrating = false;
    }
    // calculate the Rs/Ro
    float graphRatio = resistivity / memory->getValue();
    
    // plug into the graph -- mapping the values
    plugIntoGraph(graphRatio);
    
  return 0;
}
```
