# ece528L_HW1

## Section I : Review Questions
1. Hardware timers are controlled counters within a processor. They can be used to generate PWM's or control when an action should occur.

2. The primary clock signal runs the CPU.
    - Auxillary Clock - Restricted to a maximum frequency of 128 kHz. Selectable by individual peripheral modules. 
    - Master Clock - Used by the the CPU and peripheral module interfaces. Can also be used by some peripheral modules.
    - Subsystem Master Clock - Can be selected by individual peripheral modules 
    - Low-Speed Subsystem Master Clock -Uses the subsystem master clock, limited in frequency to half of the rated max frequency of Subsystem Master Clock. Can also be selected by individual peripherla modules.
    - Low-Speed Backup Domain Clock - Primarily used in the backup domain. Restriced to a max frequency of 32.768 kHz.

3. 16 bit timer => 0xFFFF = 65,535 count.

4. 24 bit timer => 0xFFFFFF = 16,777,215 \
    $16,777,215 * (1/(10 *10^6Hz)) = 1.67s$

5. SysTick timer is an integrated system timer. It's 24-bit clear-on-write dcrementing wrap-on-zero counter with a flexible control mechanism. Example of usages are, RTOS tick timer that fires at a programmable rate, A simuple counter used to measure time to completion and time used. The SysTick clock source can only be the Core Clock and is done by setting the 2nd bit in the STCSR register.

6. The SysTick has four register, STCSR, STRVR, STCVR, and STCR, described below.

    - STCSR - SysTick Control and Status Register, enables SysTick features
    - STRVR - SysTick Reload Value Register, specifies the start value to load into the current value register when the counter reaches 0.  
    - STCVR - SysTick Current Value Register, used to find the current value in the register.
    - STCR - SysTick Calbiration Value Register, used to enable software to scale to any required speed using divide and multiply. 

7. Timer_A peripheral is a 16-bit counter with up to 7 capture/compare registers. It has four operating modes, selectable clock sources, and configurable PWM outputs.

    - Stop - The timer is halted.
    - Up - The timer repeatedly counts from zero to the value of TAxCCR0.
    - Continuous - The timer repeatedly counts from zero to 0xFFFF.
    - Up/Down - The timer repeatedly counts from zero up to the value of TAxCCR0 and back down to zero.

8. 
    - TAxCTL - Used to configure the general operation of the Timer_A peripheral, Clock source selection, clock divider, timer mode selection, clear time, overflow interrupt enable.
    - TAxCCTLn - Controls the behavior of each individual capture/compare channel block within Timer_A. Has features for input capture, output compare, and interrupt generation.
    - TAxR - Displays the actual count of Timer_A
    - TAxCCRn - Each timer has multiple registers. Used for compare and capture modes.
    - TAxIV - Used for interrupts, also tells which interrupt happened
    - TAxEX0 - Allows you to further divide down the frequency of the slected clock soure.

9. A PWM is a pulse width modulated signal. The width of each pulse is a function of the amplitude of the signal.
    - LED dimming
    - Motor Control
    - Audio Genneration

10. Four Timer A modules. Two CCTL register can be used for each PWM. Each CCCTL register is treated as the duty cycle. Each TimerA module has 5 capture modules. 10 PWM's can be control if using the spare capture module correctly.

11. P2.4 must be set to its primary function. The PWM pins must be set to drive. The period register and settings for the capture/compare register

    ```
    void Timer_A0_PWM_Init(uint16_t period_constant, uint16_t duty_cycle_1, unit16_t duty_cycle_2)
    {
       if(duty_cycle_1 >= period_constant) return;
       if(duty_cycle_2 >= period_constant) return;

        // Configure pin 2.4 to use peripheral function mode
        // setting bit 4 of SEL0 register for P2
        // clearing bit 4 of SEL1 register for P2

        P2->SEL0 |= 0x10;
        P2->SEL1 &= ~0x10;

        //Configure pin 2.4 as output GPIO pins to drive the PWM signal
        P2->DIR |= 0x10;
        
        TIMER_A0->CTL |= 0x0270;
        TIMER_A0->CCTL[1] |= 0x0040;
        // Period = (2*period_constant)/(12 MHz/ prescale Value)
        // Period = 20ms
        // prescale Value = 2
        TIMER_A0->CCR[0] = 60000;

        //Duty Cycle % : duty_cycle_1 / period_contant
        // 50% => .5*60000 = 30000 = duty_cycle_1
        TIMER_A0 -> CCR[1] = 30000;
    }

    ```
12. To enable Timer_A0 as a periodic interrupt the CTL register must be set properly.

    ```
        //Setting for Timer
        TIMER_A0->CTL |= 0x0214;
        TIMER_A0->EX0 = 0x0000;
        //period = 2ms = (1*period_constant)/(12MHz/1)
        // period_constant = 24,000-1
        TIMER_A0->CCR[0] = (24000-1);

        //Setting for Interrupt
        NVIC->IP[2] = (NVIC->IP[2] & 0xFFFFF00F) | 0x00000060;
        NVIC->ISER[0] |= 0x00000100;
    ```

