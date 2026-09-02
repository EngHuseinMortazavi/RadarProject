/*******************************************************
  Radar Project - ATmega16A (UART to Processing)
*******************************************************/
#include <mega16a.h>
#include <delay.h>
#include <math.h>

// Pin Definitions
#define TRIG PORTC.7

// Global variables
unsigned long pulse_width = 0;
unsigned char edge_flag = 0;
unsigned char measurement_done = 0;
unsigned int timer0_overflow = 0;
unsigned int distance = 0;
int ang = 0;
int direction = 0;

// ==================== Ultrasonic ====================
interrupt [EXT_INT0] void ext_int0_isr(void)
{
// Place your code here
    if (edge_flag == 0) {                    // Rising edge
        TCNT0 = 0;
        timer0_overflow = 0;
        TCCR0 = (1<<CS01) | (1<<CS00);       // Prescaler 64 ? 8us per tick @ 8MHz
        MCUCR = (1<<ISC01) | (0<<ISC00);     // Switch to Falling edge
        edge_flag = 1;
    } else {                                 // Falling edge
        TCCR0 = 0;                           // Stop Timer0
        pulse_width = ((unsigned long)timer0_overflow << 8) | TCNT0;
        MCUCR = (1<<ISC01) | (1<<ISC00);     // Back to Rising edge
        edge_flag = 0;
        measurement_done = 1;
    }
}

// Timer 0 overflow interrupt service routine
interrupt [TIM0_OVF] void timer0_ovf_isr(void)
{
// Place your code here
    timer0_overflow++;
}

unsigned int get_distance(void)
{
    pulse_width = 0;
    measurement_done = 0;
    edge_flag = 0;
    timer0_overflow = 0;

    // Send Trigger Pulse
    TRIG = 0;
    delay_us(3);
    TRIG = 1;
    delay_us(12);
    TRIG = 0;

    // Wait up to 60ms for echo
    delay_ms(60);

    if (measurement_done) {
        // Combine overflow and timer value (each tick = 8us with prescaler 64)
        pulse_width = ((unsigned long)timer0_overflow << 8) | TCNT0;
        if (pulse_width < 50000) {
            return pulse_width / 58;        // Approx cm (adjusted for prescaler)
        }
    }
    
    return 0;   // Timeout or error
}
// ==================== Servo ====================
void servo_init(void) {
    DDRD |= (1<<5);
    TCCR1A = (1<<COM1A1) | (1<<WGM11);
    TCCR1B = (1<<WGM13) | (1<<WGM12) | (1<<CS11);
    ICR1H = (20000 >> 8);
    ICR1L = (unsigned char)20000;
    OCR1A = 1500;
}

void servo_write(unsigned char angle) {
    if(angle > 180) angle = 180;
    angle = 180 - angle;
    OCR1A = 1000 + ((unsigned long)angle * 1000) / 180;
}

// ==================== UART ====================
void uart_init(void) {
    UCSRB = (1<<RXEN) | (1<<TXEN);
    UCSRC = (1<<URSEL) | (1<<UCSZ1) | (1<<UCSZ0);
    UBRRL = 51;      // 9600 baud @ 8MHz
    UBRRH = 0;
}

void uart_send_char(char c) {
    while (!(UCSRA & (1<<UDRE)));
    UDR = c;
}

void uart_send_number(unsigned int num) {
    char buf[8];
    unsigned char i = 0;
    
    if (num == 0) {
        uart_send_char('0');
        return;
    }
    
    while (num > 0) {
        buf[i++] = (num % 10) + '0';
        num /= 10;
    }
    while (i > 0) {
        uart_send_char(buf[--i]);
    }
}

void uart_send_string(char *str) {
    while (*str) {
        uart_send_char(*str++);
    }
}

// ==================== Main ====================
void main(void)
{
    // Ports
    DDRC |= (1<<7);          // TRIG
    DDRD |= (1<<5);          // Servo
    PORTD |= (1<<2);           // Pull-up on Echo

    
// Timer/Counter 0 initialization
// Clock source: System Clock
// Clock value: 1000/000 kHz
// Mode: Normal top=0xFF
// OC0 output: Disconnected
// Timer Period: 0/256 ms
TCCR0 = 0x00;   // Stopped
TCNT0=0x00;
OCR0=0x00;

// Timer/Counter 1 initialization
// Clock source: System Clock
// Clock value: 1000/000 kHz
// Mode: Fast PWM top=ICR1
// OC1A output: Non-Inverted PWM
// OC1B output: Disconnected
// Noise Canceler: Off
// Input Capture on Falling Edge
// Timer Period: 1 us
// Output Pulse(s):
// OC1A Period: 1 us
// Timer1 Overflow Interrupt: Off
// Input Capture Interrupt: Off
// Compare A Match Interrupt: Off
// Compare B Match Interrupt: Off
TCCR1A=(1<<COM1A1) | (0<<COM1A0) | (0<<COM1B1) | (0<<COM1B0) | (1<<WGM11) | (0<<WGM10);
TCCR1B=(0<<ICNC1) | (0<<ICES1) | (1<<WGM13) | (1<<WGM12) | (0<<CS12) | (1<<CS11) | (0<<CS10);
TCNT1H=0x00;
TCNT1L=0x00;
ICR1H=0x00;
ICR1L=0x00;
OCR1AH=0x00;
OCR1AL=0x00;
OCR1BH=0x00;
OCR1BL=0x00;

// Timer/Counter 2 initialization
// Clock source: System Clock
// Clock value: Timer2 Stopped
// Mode: Normal top=0xFF
// OC2 output: Disconnected
ASSR=0<<AS2;
TCCR2=(0<<PWM2) | (0<<COM21) | (0<<COM20) | (0<<CTC2) | (0<<CS22) | (0<<CS21) | (0<<CS20);
TCNT2=0x00;
OCR2=0x00;

// Timer(s)/Counter(s) Interrupt(s) initialization
TIMSK=(0<<OCIE2) | (0<<TOIE2) | (0<<TICIE1) | (0<<OCIE1A) | (0<<OCIE1B) | (0<<TOIE1) | (0<<OCIE0) | (1<<TOIE0);

// External Interrupt(s) initialization
// INT0: On
// INT0 Mode: Any change
// INT1: Off
// INT2: Off
GICR|=(0<<INT1) | (1<<INT0) | (0<<INT2);
MCUCR=(0<<ISC11) | (0<<ISC10) | (1<<ISC01) | (1<<ISC00);
MCUCSR=(0<<ISC2);
GIFR=(0<<INTF1) | (1<<INTF0) | (0<<INTF2);

    // Timer1 Servo
    servo_init();

    // UART
    uart_init();

    #asm("sei")

    uart_send_string("Radar Started\r\n");

    while (1)
    {
        if(direction == 0) ang += 5;
        else ang -= 5;
        
        if(ang >= 180) direction = 1;
        if(ang <= 0)   direction = 0;
        
        servo_write(ang);
        distance = get_distance();
        
        // Send: angle,distance
        uart_send_number(ang);
        uart_send_char(',');
        uart_send_number(distance);
        uart_send_string("\r\n");
        
        delay_ms(50);
    }
}