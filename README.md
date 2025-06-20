# Soil moisture detection system
A low-cost soil moisture monitoring system built with a PIC18F45K20 microcontroller, using analog sensors and LED indicators to classify soil conditions as dry, moist, or wet.

## Features
- Real-time soil moisture sensing using homemade probes
- Visual LED indication (Red, Yellow, Green)
- Coded in C using MPLAB X and XC8
- Calibrated for dry/moist/wet conditions

## Hardware Used
- PIC18F45K20 on SEPIA board
- Breadboard and jumper wires
- M6 x 50mm screws (soil probes)
- 10kΩ and 330Ω resistors
- Red, Yellow, Green LEDs
- USB Power

## Images
![WhatsApp Image 2025-05-25 at 16 18 10_40be600e](https://github.com/user-attachments/assets/04d2efe6-aaff-46af-9639-5291587608b5)
Figure 1: Red LED lights up to indicate the soil is dry

![WhatsApp Image 2025-05-25 at 16 18 18_b2a9c610](https://github.com/user-attachments/assets/c73ab976-00fc-4173-b816-de31f9d877e0)
Figure 2: Yellow LED lights up to indicate the soil is moist

![WhatsApp Image 2025-05-25 at 16 18 21_89477995](https://github.com/user-attachments/assets/41664b31-c367-456a-8ae0-cf84ae9f6114)
![WhatsApp Image 2025-05-25 at 16 18 20_0efb68c4](https://github.com/user-attachments/assets/cd7b4e5e-9069-4d6f-adc6-8d7edb03990f)
Figure 3 and 4: Green LED lights up to indicate the soil is moist



## Code
/* 
 * EPJ 2A - Soil Moisture Detector
 * Student: JAHANVI SINGH, ID: 224080807
 * 
 * Hardware Setup:
 * - PIC18F45K20 MCU @ 8MHz
 * - Soil Sensor: 2 nails + 10k ohm resistor
 * - LEDs: Red(RC0), Yellow(RC1), Green(RC2) with 330 ohm resistors
 */

#include <xc.h>

// CORRECT Configuration Bits for PIC18F45K20
#pragma config FOSC = INTIO67   // Internal oscillator
#pragma config WDTEN = OFF      // Watchdog Timer disabled
#pragma config LVP = OFF        // Low-voltage programming disabled
#pragma config CP0 = OFF        // Code protection disabled

#define _XTAL_FREQ 8000000      // 8MHz clock for delays

// Calibration Values (adjust based on testing)
#define DRY_THRESHOLD   900     // ADC > 900 = Dry
#define WET_THRESHOLD   350     // ADC < 350 = Wet

void initialize() {
    // Clock Setup
    OSCCON = 0x70;              // 8MHz internal oscillator
    
    // Pin Directions
    TRISAbits.TRISA0 = 1;       // RA0 as input (soil sensor)
    TRISC = 0x00;               // All PORTC as output (LEDs)
    LATC = 0x00;                // Turn off all LEDs
    
    // ADC Configuration
    ADCON0 = 0x01;              // ADC ON, Channel 0 (AN0)
    ADCON1 = 0x0E;              // AN0 analog, others digital
    ADCON2 = 0xA5;              // Right justify, Fosc/8, 12TAD
}

unsigned int read_soil() {
    ADCON0bits.GO = 1;          // Start conversion
    while(ADCON0bits.GO);       // Wait for completion
    return ((unsigned int)(ADRESH << 8)) | ADRESL; // Fixed warning
}

void update_leds(unsigned int moisture) {
    if(moisture > DRY_THRESHOLD) {
        LATC = 0b00000001;      // Red ON (Dry)
    } 
    else if(moisture < WET_THRESHOLD) {
        LATC = 0b00000100;      // Green ON (Wet)
    } 
    else {
        LATC = 0b00000010;      // Yellow ON (Moist)
    }
}

void main(void) {
    initialize();
    #pragma config DEBUG = ON  // Enable debug mode
    
    while(1) {
        unsigned int moisture = read_soil();
        update_leds(moisture);
        __delay_ms(500);        // 0.5s delay
    }
}


