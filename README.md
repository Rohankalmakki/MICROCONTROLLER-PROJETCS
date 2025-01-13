//SMART HOME AUTOMATION SYSTEM FOR LIGHT & TEMPERATURE CONTROL





#include<lpc21xx.h>
#include "lcd_fun.c"
#include "uart.h"
 
void config_LED_pin();
void config_FAN_pin();
void config_TempSensorToADC();
void config_PWM();  
int read_temp_value();
void config_LightSensorToADC();
int read_light_value();
void pll_12MHz(void);
void pll_36MHz(void);
void pll_48MHz(void);
void pll_60MHz(void); 
int main()
{
unsigned int temperature;
unsigned int LightValue;
lcd_config();
uart_config();
config_FAN_pin();
config_LED_pin();
config_PWM();
while(1)
{
lcd_cmd(0x01);	
temperature=read_temp_value();		  
if(temperature<26)
{
PWMMR4=0;
PWMLER=(1<<4);
IOSET0=(1<<2);
delay(500);
IOCLR0=(1<<2);
delay(500);
lcd_cmd(0x80);
lcd_str("Temp is ");
uart_str("Temperature is ");
lcd_num(temperature);
uart_num(temperature);
lcd_str("oC");
lcd_cmd(0xC0);
lcd_str("FAN IS OFF");
uart_str("FAN IS OFF");
delay(1000);
lcd_cmd(0x01);
}
else if(temperature==26)
{
       	PWMMR4=5000;
    	PWMLER=(1<<4);
IOSET0=(1<<2);
delay(500);
IOCLR0=(1<<2);
delay(500);
lcd_cmd(0x80);
lcd_str("Temp is ");
uart_str("Temperature is ");
lcd_num(temperature);
uart_num(temperature);
lcd_str("oC");
lcd_cmd(0xC0);
lcd_str("FAN SPEED = 1");
uart_str("FAN SPEED = 1");
delay(1000);
lcd_cmd(0x01);
}
else if((temperature>=27)&&(temperature<=32))
{
PWMMR4=7500;
    	PWMLER=(1<<4);
IOSET0=(1<<2);
delay(500);
IOCLR0=(1<<2);
delay(500);
lcd_cmd(0x80);
lcd_str("Temp is ");
uart_str("Temperature is ");
lcd_num(temperature);
uart_num(temperature);
lcd_str("oC");
lcd_cmd(0xC0);
lcd_str("FAN SPEED = 2");
uart_str("FAN SPEED = 2");
delay(1000);
lcd_cmd(0x01);
}
else
{
PWMMR4=10000;
    	PWMLER=(1<<4);
IOSET0=(1<<2);
delay(500);
IOCLR0=(1<<2);
delay(500);
lcd_cmd(0x01);
lcd_cmd(0x80);
lcd_str("Temp is ");
uart_str("Temperature is ");
lcd_num(temperature);
uart_num(temperature);
lcd_str("oC");
lcd_cmd(0xC0);
lcd_str("FAN SPEED = 3");
uart_str("FAN SPEED = 3");
delay(1000);
lcd_cmd(0x01);	
}
ADDR&=~(1<<31);          //clear ADDR flag
delay(500);
LightValue=read_light_value();
if(LightValue<=300)
{
lcd_cmd(0x80);
lcd_str("Light Val is ");
uart_str("Light value is ");
lcd_num(LightValue);
uart_num(LightValue);
lcd_cmd(0xC0);
lcd_str("LEDs ON at 12MHz");
uart_str("LEDs ON at 12MHz");
delay(500);
IOSET1=(0xFF<<17);
delay(500);
IOCLR1=(0xFF<<17);
delay(500);
pll_12MHz();
lcd_cmd(0x01);
}
else if((LightValue>300)&(LightValue<=900))
{
lcd_cmd(0x80);
lcd_str("Light Val is ");
uart_str("Light value is ");
lcd_num(LightValue);
uart_num(LightValue);
lcd_cmd(0xC0);
lcd_str("LEDs ON at 36MHz");
uart_str("LEDs ON at 36MHz");
delay(500);
IOSET1=(0xFF<<17);
delay(500);
IOCLR1=(0xFF<<17);
delay(500);
pll_36MHz();
lcd_cmd(0x01);
}
else if((LightValue>900)&(LightValue<=1500))
{
lcd_cmd(0x80);
lcd_str("Light is ");
uart_str("Light value is ");
lcd_num(LightValue);
uart_num(LightValue);
lcd_cmd(0xC0);
lcd_str("LEDs ON at 48MHz");
uart_str("LEDs ON at 48MHz");
delay(500);
IOSET1=(0xFF<<17);
delay(500);
IOCLR1=(0xFF<<17);
delay(500);
pll_48MHz();
lcd_cmd(0x01);
}
else
{
lcd_cmd(0x80);
lcd_str("Light is ");
uart_str("Light value is ");
lcd_num(LightValue);
uart_num(LightValue);
lcd_cmd(0xC0);
lcd_str("LEDs ON at 60MHz");
uart_str("LEDs ON at 60MHz");
IOSET1=(0xFF<<17);
delay(1000);
IOCLR1=(0xFF<<17);
delay(1000);
pll_60MHz();
lcd_cmd(0x01);
}
lcd_cmd(0x01);			 //cmd to clear display
}
}
 
void config_FAN_pin()
{
/*Configure P0.8 as PWM4*/
   PINSEL0|=(1<<17);
   PINSEL0&=~(1<<16);
}
void config_LED_pin()
{
IODIR1|=(0xFF<<17);  //Configure all LEDs as Output port
}
void config_TempSensorToADC()
{
//Configure P0.28 as AIN1
PINSEL1 |=(1<<24);		  //24=1
PINSEL1&=~(1<<25);		  //25=0
}
void config_LightSensorToADC()
{
//Configure P0.29 as AIN2
PINSEL1 |=(1<<26);		  //26=1
PINSEL1&=~(1<<27);		  //27=0
}
void config_PWM()
{
/*PWM Configuration*/
   PWMPR=14;	     //Load 14 to PR
   PWMMR0=10000;	 //Load total time period to MR0
   PWMLER=(1<<0);	 //Lock and enable the value in MR0
   PWMMCR=(1<<1);
   PWMPCR&=(1<<4);
   PWMPCR|=(1<<12);
   PWMTCR|=(1<<0);
   PWMTCR|=(1<<3);
}
int read_temp_value()
{
unsigned int val;
config_TempSensorToADC();
//Configure ADC-->Sel CH-1, CLKDIV=4,Burst mode, PDN in operational mode
ADCR=(1<<1) | (4<<8) | (1<<16) | (1<<21);
while(!(ADDR&(1<<31)));	  // monitor EOC using polling method
delay(1000);             
val=ADDR&(0x3FF<<6);	 // extract 10 bit digital data from RESULT
val=val>>6;				 // right shift to get as LSB
val=val/3.3;			 //Temp value in degree Celsius
return val;
}
int read_light_value()
{
unsigned int val;
config_LightSensorToADC();
//Configure ADC-->Sel CH-2, CLKDIV=4,Burst mode, PDN in operational mode
ADCR=(1<<2) | (4<<8) | (1<<16) | (1<<21);
while(!(ADDR&(1<<31)));	  // monitor EOC using polling method
delay(1000);             
val=ADDR&(0x3FF<<6);	 // extract 10 bit digital data from RESULT
val=val>>6;
//val=val/10;				 // right shift to get as LSB
return val;
}
void pll_12MHz(void)
{
//Configure M=1, MSEL=00000, Bit[4:0], P=8, PSEL=11, Bit[6:5]
PLLCFG=(1<<5)|(1<<6);
PLLCON=(1<<0);	                 //Enable PLL
PLLFEED=0xAA;	   	             //Triggering signal
PLLFEED=0x55;
while(!(PLLSTAT & (1<<10)));	//Monitor Phase Lock Condition using polling method
PLLCON=(1<<0)|(1<<1);			//Enable & connect PLL
PLLFEED=0xAA;
PLLFEED=0x55;
}
void pll_36MHz(void)
{
//Configure M=3, MSEL=00010, Bit[4:0], P=4, PSEL=, Bit[6:5]
PLLCFG=(1<<1)|(1<<6);
PLLCON=(1<<0);	
PLLFEED=0xAA;	   	
PLLFEED=0x55;
while(!(PLLSTAT & (1<<10)));	
PLLCON=(1<<0)|(1<<1);			
PLLFEED=0xAA;
PLLFEED=0x55;
}
void pll_48MHz(void)
{
//Configure M=4, MSEL=00011, Bit[4:0], P=4, PSEL=10, Bit[6:5]
PLLCFG=(1<<1)|(1<<0)|(1<<6);
PLLCON=(1<<0);	
PLLFEED=0xAA;	   	
PLLFEED=0x55;
while(!(PLLSTAT & (1<<10)));	
PLLCON=(1<<0)|(1<<1);			
PLLFEED=0xAA;
PLLFEED=0x55;
}
void pll_60MHz(void)
{
//Configure M=5, MSEL=00100, Bit[4:0], P=2, PSEL=01, Bit[6:5]
PLLCFG=(1<<3)|(1<<5);
PLLCON=(1<<0);	
PLLFEED=0xAA;	   
PLLFEED=0x55;
while(!(PLLSTAT & (1<<10)));	
PLLCON=(1<<0)|(1<<1);			
PLLFEED=0xAA;
PLLFEED=0x55;
}
