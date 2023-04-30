#include <msp430.h>

#define SERVO_PIN BIT0
#define BUTTON_LEFT BIT1
#define BUTTON_RIGHT BIT2
#define PWM_PERIOD 20000
#define SERVO_MAX_DEGREE 180
#define SERVO_MIN_PULSE_WIDTH 1000 // microseconds
#define SERVO_MAX_PULSE_WIDTH 2000 // microseconds

int pulse_width = SERVO_MIN_PULSE_WIDTH; // start at minimum pulse width

void main(void) {
    WDTCTL = WDTPW + WDTHOLD; // disable watchdog timer
    P1DIR |= SERVO_PIN; // set SERVO_PIN as output
    P1SEL |= SERVO_PIN; // enable PWM output on SERVO_PIN
    TA0CCR0 = PWM_PERIOD - 1; // set PWM period
    TA0CCTL1 = OUTMOD_7; // set PWM output mode
    TA0CCR1 = pulse_width; // start at minimum pulse width
    TA0CTL = TASSEL_2 + MC_1 + TACLR; // use SMCLK, up mode, clear timer
    P1DIR &= ~(BUTTON_LEFT + BUTTON_RIGHT); // set BUTTON_LEFT and BUTTON_RIGHT as input
    P1REN |= BUTTON_LEFT + BUTTON_RIGHT; // enable pull-up resistors on BUTTON_LEFT and BUTTON_RIGHT
    P1OUT |= BUTTON_LEFT + BUTTON_RIGHT; // set initial value of BUTTON_LEFT and BUTTON_RIGHT to high

    while (1) {
        if (!(P1IN & BUTTON_LEFT)) { // if BUTTON_LEFT is pressed
            if (pulse_width > SERVO_MIN_PULSE_WIDTH) { // if pulse width is not at minimum value
                pulse_width -= 100; // decrease pulse width by 100 microseconds
                TA0CCR1 = pulse_width; // update pulse width
            }
            __delay_cycles(10000); // debounce delay
        } else if (!(P1IN & BUTTON_RIGHT)) { // if BUTTON_RIGHT is pressed
            if (pulse_width < SERVO_MAX_PULSE_WIDTH) { // if pulse width is not at maximum value
                pulse_width += 100; // increase pulse width by 100 microseconds
                TA0CCR1 = pulse_width; // update pulse width
            }
            __delay_cycles(10000); // debounce delay
        }
    }
}

