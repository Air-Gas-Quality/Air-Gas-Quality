# Materials : 

## MQ-135 and other gas sensors :
- [MQ-135 Gas Sensor Pinout, Features, Alternatives, Datasheet & Uses Guide.pdf](https://github.com/Air-Gas-Quality/Air-Gas-Quality/files/8318365/MQ-135.Gas.Sensor.Pinout.Features.Alternatives.Datasheet.Uses.Guide.pdf)
- https://www.electronicoscaldas.com/datasheet/MQ-135_Hanwei.pdf
- https://ram-e-shop.com/?s=MQ&product_cat=0&post_type=product

## Environmental Analysis : 
- https://www.airnow.gov/aqi-and-health/
- [aqi-technical-assistance-document-sept2018.pdf](https://github.com/Air-Gas-Quality/Air-Gas-Quality/files/8318367/aqi-technical-assistance-document-sept2018.pdf)

## MQ-135 Cracking Datasheet : 

1) Basic Functionality : 
---> Potentiometer (SnO2 -- Semiconductor). 
---> Gases ---> Voltage increases/Resistivity decreases.


2) What we want to find ? 
- We want to find the Rs (Sensitivity resistance) in the fresh clean air (calibration) and 
in various gaseous statuses.
- Then, we want to find the ratio `Rs/Ro`; where `Rs : Sensitivity Resistance in various gaseous statuses`, `Ro = Original Resistance in fresh air`.
- Then, we plug the ratio into the graph from the datasheet to find the possible corresponding gas : 

![MQ-135](https://user-images.githubusercontent.com/60224159/159438596-55ff8d4f-c548-4b93-8070-c7d9e8996670.png)

- Example : 
### Recall; `Rs/Ro` = 0.9;
### then : 
The Air Quality state is `moderate` and most probable harmful gases that may lead to this resistivity are : 
1) Acetona @ 50 ppm.
2) Toleuon @ 70 ppm.
3) Alcohol @ 100 ppm.
4) Ammonia @ 100-150 ppm.
5) Co2 @ 150-180 ppm.

3) Suggested Code : 
```c
bool isCalibrating = true;
const Memory* memory = new Memory();
// max voltage 
static const float MAX_VOLTAGE = 5.0;

float getResistivity(float current) {
      // analogValue = [0, 1023] (Decimal Encoded value) = [0V, 5.0V] (Real Voltage)
      voltaile uint10_t potentiometerValue = analogRead(A0); // 5 (0000000101)

      // convert to percentage 
      const float rationalValue = potentiometerValue / 1023f; 

      // convert to voltage 
      const float conductivity = rationalValue * MAX_VOLTAGE;  

      // get resistivity
      const float resistivity = conductivity / current;   
}

int main() {
    float resistivity = getResistivity(1);
    // save the calibration value
    if (isCalibrating && !(memory->hasData())) {
      memory->save(resistivity);
    }
    // calculate the Rs/Ro
    float graphRatio = resistivity / memory->getValue();
    
    // plug into the graph -- mapping the values
    plugIntoGraph(graphRatio);
    
  return 0;
}
```


