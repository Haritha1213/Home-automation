# Home-automation
**Home automation using AVR Atmega32 microcontroller**

**AVR ATMEGA32 CODE**
'''
#include <avr/io.h>
#define F_CPU 16000000UL
#include <util/delay.h>
#define LCD PORTD
#define EN 2
#define RW 1
#define RS 0
void lcdcmd(unsigned char cm)
{
	PORTC &=~(1<<RS);  //RS = 0 - command
	PORTC &=~(1<<RW); //RW = 0 - write to lcd
	PORTC|=(1<<EN);  //high to low pulse is given to en - to latch data
	LCD=cm;
	_delay_ms(1);
	PORTC &=~(1<<EN); 
	
}
void lcddata(unsigned char dat)
{
	PORTC |=(1<<RS);  //RS = 1 - data
	PORTC&=~(1<<RW);  //RW = 0 - write to lcd
	PORTC|=(1<<EN);  //high to low pulse is given to en - to latch data
	LCD=dat;
	_delay_ms(1);
	PORTC &=~(1<<EN);
	
}
void init() {
	DDRC=0XFF;  //o/p - connected to lcd
	DDRD=0XFF;  //o/p - transfer data or cmds to lcd
	lcdcmd(0X38); //8 BIT 2 LINES
	lcdcmd(0X0C); //DISPLAY ON CURSOR ON 
	//lcdcmd(0X01); //clear screen
	lcdcmd(0X80); //ROW 0 COLUMN 0
	_delay_ms(1);
}

//fn to display string
void lcdprint(char *str){
	unsigned char i=0;
	while(str[i]!=0){
		lcddata(str[i]);
		i++;
	}
}

//bcd to ascii conversion
void convert(unsigned char val){
	unsigned char x,y,d1,d2;
	
	//for bcd output
	/*
	x = val & 0x0F;
	d1 = x;
	y = val & 0xF0;
	d2 = (y>>4);  */

	
	 //for binary output  
	 d1=val/10;      //d2 = val & 0xF0  and d2 = d2 >> 4
	 
	 d2=val%10;  // d1 = val & 0x0F
	
	lcdcmd(0xCD);
	lcddata(d1  | 0x30);  //d1 | 0x30
	lcddata(d2 | 0x30);  //d2 | 0x30 
}


int main(void)
{  
	
    DDRA &= 0b11111100;   //i/p A0 - temperature sensor and A1 - ir sensor
	DDRB |= 0b00000011; //o/p LED  and motor
	
	unsigned char x;
	unsigned char data;
	
	ADCSRA = 0x87;  // 1000 0111
	ADMUX = 0xE0; //2.56 V  1110 0000
	 
	
    while (1) 
    {
		init();
		lcdprint("LED OFF");
		lcdcmd(0xC0);
		lcdprint("Temperature");
		
		x = PINA; //read sensor i/p for ir sensor (digital)
		
		 ADCSRA |= (1<<ADSC);  //set ADSC = 1
		 while((ADCSRA&(1<<ADIF))==0); //when ADIF = 0, do nothing...stay in the loop.
		 data = ADCH;  
		 convert(data);
		 
		 //if IR o/p = 1
		if(x & 0b00000010)
		{
			//when obstacle detected: PC.3 is 1
			//ON the led
			PORTB |= 0b00000001; 
			lcdcmd(0x01);
			lcdprint("LED ON");
			//temp > 30
			 if(data>30){
				 PORTB |= 0b00000010;
				 lcdcmd(0xC0);
				 lcdprint("DCM ON");
			 }
			_delay_ms(70);
         }

	 else{
		 //OFF led and motor
		 PORTB &= 0x00;
		 			
	 }

	}
	 
	 return 0;
}
'''
