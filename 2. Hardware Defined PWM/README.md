#include <msp430.h>

#define PWM_PERIOD 1000 // 1ms period
#define COLORS 6 // number of colors to cycle through
#define COLOR_STEPS 50 // number of steps for each color transition

int current_color = 0;
int current_step = 0;
int r = 0, g = 0, b = 0;

int colors[COLORS][3] = {
  {255, 0, 0},   // red
  {255, 128, 0}, // orange
  {0, 255, 0},   // green
  {0, 255, 255}, // cyan
  {0, 0, 255},   // blue
  {128, 0, 255}  // purple
};

void setup_pwm() {
  TA0CTL = TASSEL_2 + MC_1 + ID_0; // SMCLK, Up mode, divide by 1
  TA0CCR0 = PWM_PERIOD - 1; // set PWM period
  TA0CCTL1 = OUTMOD_7; // reset/set output mode for CCR1
  TA0CCTL2 = OUTMOD_7; // reset/set output mode for CCR2
  TA0CCTL3 = OUTMOD_7; // reset/set output mode for CCR3
}

void update_color() {
  int next_color = (current_color + 1) % COLORS;
  float r_step = (float)(colors[next_color][0] - colors[current_color][0]) / COLOR_STEPS;
  float g_step = (float)(colors[next_color][1] - colors[current_color][1]) / COLOR_STEPS;
  float b_step = (float)(colors[next_color][2] - colors[current_color][2]) / COLOR_STEPS;
  r += r_step;
  g += g_step;
  b += b_step;
  current_step++;
  if (current_step >= COLOR_STEPS) {
    current_step = 0;
    current_color = next_color;
  }
}

void set_rgb(int r_val, int g_val, int b_val) {
  TA0CCR1 = r_val;
  TA0CCR2 = g_val;
  TA0CCR3 = b_val;
}

int main(void) {
  WDTCTL = WDTPW + WDTHOLD; // stop watchdog timer
  P6SEL |= BIT0 | BIT1 | BIT2; // set P6.0, P6.1, P6.2 as Timer_A CCR output
  P6DIR |= BIT0 | BIT1 | BIT2; // set P6.0, P6.1, P6.2 as output
  setup_pwm();

  while (1) {
    update_color();
    set_rgb(r, g, b);
    __delay_cycles(10000); // wait a bit between color updates
  }
}
