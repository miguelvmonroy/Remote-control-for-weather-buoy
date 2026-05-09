#include <msp430.h>
#include <stdint.h>

/*
CONEXIONES DE PUERTOS:
Port 1:
P1.0 → Salida LED Boya
P1.1 → Entrada Botón Boya
P1.2 → Salida LED Hallow
P1.5 → Entrada LED Hallow

Port 2:
P2.0 → MCLK (reloj)
P2.2 → ACLK (reloj)

Port 5:
P5.2 → Entrada LED Boya

Port 7:
P7.4 → Pulso B
P7.5 → Pulso A
P7.7 → SMCLK (reloj)
*/

// ===========================================================================
// DEFINICIONES DE PUERTOS Y PINES
// ===========================================================================

// Puerto 1
#define PIN_TX_CONDICION_LED_BOYA               BIT0    // P1.0 → Salida LED Boya       
#define PIN_BOTON_ENCENDER_APAGAR_BOYA          BIT1    // P1.1 → Entrada Botón Boya    
#define PIN_TX_CONDICION_LED_HALLOW             BIT2    // P1.2 → Salida LED Hallow     
#define PIN_LED_HALLOW_ENCENDIDA_APAGADA        BIT5    // P1.5 → Entrada LED Hallow    

// Puerto 2
#define PIN_MCLK_SALIDA                         BIT0    // P2.0 → MCLK (reloj)
#define PIN_ACLK_SALIDA                         BIT2    // P2.2 → ACLK (reloj)

// Puerto 5
#define PIN_LED_BOYA_ENCENDIDA_APAGADA          BIT2    // P5.2 → Entrada LED Boya          

// Puerto 7
#define PIN_GENERA_PULSO_B                      BIT4    // P7.4 → Pulso B         
#define PIN_GENERA_PULSO_A                      BIT5    // P7.5 → Pulso A         
#define PIN_SMCLK_SALIDA                        BIT7    // P7.7 → SMCLK (reloj)

// ===========================================================================
// VARIABLES GLOBALES
// ===========================================================================
volatile unsigned int Bandera = 0;          // Control de estado (0=Hallow, 1=Boya)

// ===========================================================================
// 1. CONFIGURACIÓN DEL RELOJ (25 MHz)
// ===========================================================================


void SetVcoreUp (unsigned int level)
{
    PMMCTL0_H = PMMPW_H;              // Unlock PMM

    PMMCTL0_L = (PMMCTL0_L & ~PMMCOREV_3) | level;

    PMMCTL0_H = 0;                    // Lock PMM
}


void Config_Reloj(void)
{
    // Detener el Watchdog Timer
    WDTCTL = WDTPW | WDTHOLD;
    
    SetVcoreUp(PMMCOREV_1);
    SetVcoreUp(PMMCOREV_2);
    SetVcoreUp(PMMCOREV_3);


    // Cristal externo = 32,768 Hz
    
    // --- CONFIGURACIÓN DEL RELOJ A 25 MHz ---
    UCSCTL3 = SELREF_2;
    UCSCTL4 |= SELA_2;
    __bis_SR_register(SCG0);
    UCSCTL0 = 0x0000;
    UCSCTL1 = DCORSEL_7;
    UCSCTL2 = FLLD_0 + 762;
    __bic_SR_register(SCG0);

    do {
        UCSCTL7 &= ~(XT2OFFG + XT1LFOFFG + DCOFFG);
        SFRIFG1 &= ~OFIFG;
    } while (SFRIFG1 & OFIFG);
    __delay_cycles(1000000);

    // --- SALIDAS DE RELOJ PARA MEDICIÓN  ---
    P2DIR |= PIN_ACLK_SALIDA;
    P2SEL |= PIN_ACLK_SALIDA;
    P7DIR |= PIN_SMCLK_SALIDA;
    P7SEL |= PIN_SMCLK_SALIDA;
    P2DIR |= PIN_MCLK_SALIDA;
    P2SEL |= PIN_MCLK_SALIDA;

    UCSCTL4 = SELA_2 | SELM_3 | SELS_3;

}

void Config_Reloj_1M(void)
{
    // Detener el Watchdog Timer
    WDTCTL = WDTPW | WDTHOLD;

    // Cristal externo = 32,768 Hz
    
    // --- CONFIGURACIÓN DEL RELOJ A 25 MHz ---
    UCSCTL3 = SELREF_2;
    UCSCTL4 |= SELA_2;
    __bis_SR_register(SCG0);
    UCSCTL0 = 0x0000;
    
    //UCSCTL2 = FLLD_0 + 762;   // a 25 MHZ 
    UCSCTL2 = FLLD_0 + 30;   // a 1 MHZ 
    UCSCTL1 = DCORSEL_2;
    __bic_SR_register(SCG0);
    __delay_cycles(100000);
    
    do {
        UCSCTL7 &= ~(XT2OFFG + XT1LFOFFG + DCOFFG);
        SFRIFG1 &= ~OFIFG;
    } while (SFRIFG1 & OFIFG);
   

    // --- SALIDAS DE RELOJ PARA MEDICIÓN  ---
    P2DIR |= PIN_ACLK_SALIDA;
    P2SEL |= PIN_ACLK_SALIDA;
    P7DIR |= PIN_SMCLK_SALIDA;
    P7SEL |= PIN_SMCLK_SALIDA;
    P2DIR |= PIN_MCLK_SALIDA;
    P2SEL |= PIN_MCLK_SALIDA;

    UCSCTL4 = SELA_2 | SELM_3 | SELS_3;

}


// ===========================================================================
// 2. CONFIGURACIÓN DE PUERTOS
// ===========================================================================
void Config_Puertos(void)
{
    // P1.5 como entrada (Lee P1.5 - Entrada LED Hallow)
    P1DIR &= ~PIN_LED_HALLOW_ENCENDIDA_APAGADA;
    P1REN |= PIN_LED_HALLOW_ENCENDIDA_APAGADA;
    P1OUT |= PIN_LED_HALLOW_ENCENDIDA_APAGADA;
    
    // P1.1 como entrada (Lee P1.1 - Entrada Botón encendido Boya)
    P1DIR &= ~PIN_BOTON_ENCENDER_APAGAR_BOYA;
    P1REN |= PIN_BOTON_ENCENDER_APAGAR_BOYA;
    P1OUT |= PIN_BOTON_ENCENDER_APAGAR_BOYA;
    
    // P5.2 como entrada (Lee P5.2 - Entrada LED Boya)
    P5DIR &= ~PIN_LED_BOYA_ENCENDIDA_APAGADA;
    P5REN |= PIN_LED_BOYA_ENCENDIDA_APAGADA;
    P5OUT |= PIN_LED_BOYA_ENCENDIDA_APAGADA;
    
    // P1.2 como salida (Salida LED Hallow)
    P1DIR |= PIN_TX_CONDICION_LED_HALLOW;
    P1OUT &= ~PIN_TX_CONDICION_LED_HALLOW;
    
    // P1.0 como salida (Salida LED Boya)
    P1DIR |= PIN_TX_CONDICION_LED_BOYA;
    P1OUT &= ~PIN_TX_CONDICION_LED_BOYA;
    
    // P7.5 como salida (Pulso A)
    P7DIR |= PIN_GENERA_PULSO_A;
    P7OUT &= ~PIN_GENERA_PULSO_A;
    
    // P7.4 como salida (Pulso B)
    P7DIR |= PIN_GENERA_PULSO_B;
    P7OUT &= ~PIN_GENERA_PULSO_B;
}

// ===========================================================================
// 3. PROGRAMA PRINCIPAL (sin interrupciones)
// ===========================================================================
int main(void)
{
    // 1. INICIAR RELOJ
    Config_Reloj_1M();

    // 2. INICIAR PUERTOS
    Config_Puertos();

    // (Timer e interrupciones deshabilitados)

    // 3. BUCLE PRINCIPAL - contiene la lógica que antes estaba en la ISR
    while(1)
    {
        // --- CONTROL DE LED HALLOW ---
        if (P1IN & PIN_LED_HALLOW_ENCENDIDA_APAGADA) {
            P1OUT &= ~PIN_TX_CONDICION_LED_HALLOW;   // Apagar LED Hallow
        } else {
            P1OUT |= PIN_TX_CONDICION_LED_HALLOW;    // Encender LED Hallow
        }
        
        // --- CONTROL DE PULSOS A/B (botón Boya) ---
        if (P1IN & PIN_BOTON_ENCENDER_APAGAR_BOYA) {
            // Botón no presionado: apagar ambos pulsos
            P7OUT &= ~PIN_GENERA_PULSO_B;
            P7OUT &= ~PIN_GENERA_PULSO_A;
        } else {
            // Botón presionado: alternar pulsos según Bandera
            if (Bandera == 0) {
                P7OUT &= ~PIN_GENERA_PULSO_A;
                P7OUT |= PIN_GENERA_PULSO_B;   // Activar pulso B
                Bandera = 1;
                __delay_cycles(50000);         // Retardo de 1 ms (25 MHz)
            } else {
                P7OUT &= ~PIN_GENERA_PULSO_B;
                P7OUT |= PIN_GENERA_PULSO_A;   // Activar pulso A
                Bandera = 0;
                __delay_cycles(50000);         // Retardo de 1 ms (25 MHz)
            }
        }
        
        // --- CONTROL DE LED BOYA ---
        if (P5IN & PIN_LED_BOYA_ENCENDIDA_APAGADA) {
            P1OUT |= PIN_TX_CONDICION_LED_BOYA;    // Encender LED Boya
        } else {
            P1OUT &= ~PIN_TX_CONDICION_LED_BOYA;   // Apagar LED Boya
        }
    }
}
