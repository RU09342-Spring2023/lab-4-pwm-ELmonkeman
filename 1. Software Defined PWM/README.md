#include <msp430.h>

// Constants for PWM period and duty cycle increments
#define PWM_PERIOD 1000  // 1kHz PWM frequency
#define DUTY_CYCLE_INC 100  // Increase duty cycle by 10%

// Global variables to hold duty cycle values
unsigned int duty_cycle_led1 = 500;  // 50% duty cycle
unsigned int duty_cycle_led2 = 500;  // 50% duty cycle

int main(void) {
    WDTCTL = WDTPW | WDTHOLD;  // Stop watchdog timer

    // Set up P1.0 and P6.6 as output pins for LEDs
    P1DIR |= BIT0;
    P6DIR |= BIT6;

    // Set up P2.1 and P4.3 as input pins for buttons
    P2DIR &= ~BIT1;
    P4DIR &= ~BIT3;
    P2REN |= BIT1;
    P4REN |= BIT3;
    P2OUT |= BIT1;
    P4OUT |= BIT3;

    // Configure timer A0 for PWM
    TA0CTL = TASSEL_2 | MC_1 | ID_0;  // SMCLK, up mode, no prescaler
    TA0CCR0 = PWM_PERIOD - 1;  // PWM period
    TA0CCTL1 = OUTMOD_7;  // Reset/Set output mode for CCR1 (LED 1.0)
    TA0CCR1 = duty_cycle_led1;  // Initial duty cycle for LED 1.0
    TA0CCTL2 = OUTMOD_7;  // Reset/Set output mode for CCR2 (LED 6.6)
    TA0CCR2 = duty_cycle_led2;  // Initial duty cycle for LED 6.6

    // Infinite loop
    while (1) {
        // Check if button 2.1 is pressed
        if ((P2IN & BIT1) == 0) {
            // Increase duty cycle for LED 1.0
            duty_cycle_led1 += DUTY_CYCLE_INC;
            if (duty_cycle_led1 > 1000) {
                duty_cycle_led1 = 0;
            }
            TA0CCR1 = duty_cycle_led1;  // Set new duty cycle
            __delay_cycles(100000);  // Debounce button press
        }
        // Check if button 4.3 is pressed
        if ((P4IN & BIT3) == 0) {
            // Increase duty cycle for LED 6.6
            duty_cycle_led2 += DUTY_CYCLE_INC;
            if (duty_cycle_led2 > 1000) {
                duty_cycle_led2 = 0;
            }
            TA0CCR2 = duty_cycle_led2;  // Set new duty cycle
            __delay_cycles(100000);  // Debounce button press
        }
    }
}
