# Cse347-project
#include "tm4c123gh6pm.h"
#include <stdio.h>
#include <stdint.h>
#include <FreeRTOS.h>
#include <task.h>
#include <queue.h>
#include <semphr.h>


void PortF_Init(void);
void PortB_Init(void);
xSemaphoreHandle xBinarySemaphore1;
xSemaphoreHandle xBinarySemaphore2;


void Init_portF (void)
{

  
  SYSCTL->RCGCGPIO |= 0x00000020;    // 1) F clock
  GPIOF->LOCK = 0x4C4F434B;  				 // 2) unlock PortF PF0  
  GPIOF->CR = 0x1F;          				 // allow changes to PF4-0       
  GPIOF->DIR = 0x0E;         				 // 5) PF4,PF0 input, PF3,PF2,PF1 output   
   GPIOF->PUR = 0x11;       				   // enable pullup resistors on PF4,PF0       
  GPIOF->DEN = 0x1F;       				   // 7) enable digital pins PF4-PF0
	GPIOF->DATA = 0x00;
   
}


void Init_portB (void)
{

  
  SYSCTL->RCGCGPIO |= 0x00000002;    // 1) B clock
 
  GPIOB->CR = 0xFF;          				 // allow changes to PB4-0       
   GPIOB->DIR = 0x00;         				 // 5) PB4,PB0 input, PB3,PB2,PB1 input
  GPIOF->AFSEL = 0x00;      				 // 6) no alternate function
  GPIOF->PUR = 0xFF;       				   // enable pullup resistors on PB4-->PB0       
  GPIOF->DEN = 0xFF;       				   // 7) enable digital pins PB4-PB0
	GPIOF->DATA = 0x00;
   
}



int BTN_PRESS1 (void)
{
  if ((GPIO_PORTB_DATA_R & 0x10)== 0)   //PB4
    return 1;
  else 
    return 0;
}


int BTN_PRESS2 (void)
{
  if ((GPIO_PORTB_DATA_R & 0x02)== 0)   //PB1
    return 1;
  else 
    return 0;
}




int BTN_PRESS3 (void)
{
  if ((GPIO_PORTF_DATA_R & 0x10)== 0)   //PF4
    return 1;
  else 
    return 0;
}


int BTN_PRESS4 (void)
{
  if ((GPIO_PORTB_DATA_R & 0x80)== 0)   //PB7
    return 1;
  else 
    return 0;
}



int Limit_PRESS3 (void)
{
  if ((GPIO_PORTB_DATA_R & 0x04)== 0)   //PB2
    return 1;
  else 
    return 0;
}


int Limit_PRESS4 (void)
{
  if ((GPIO_PORTB_DATA_R & 0x08)== 0)   //PB3
    return 1;
  else 
    return 0;
}


int locker(void)
{
  if ((GPIO_PORTB_DATA_R & 0x20)== 0)   //PB6
    return 1;
  else 
    return 0;
}



void Red_On (void)

  {     
       GPIO_PORTF_DATA_R |= 0X02;
  }
  
 

void Red_Off (void)

  {       
       GPIO_PORTF_DATA_R &= ~0X02;   
  }


void Blue_On (void)

  {     
       GPIO_PORTF_DATA_R |= 0X04;
  }
  
 

void Blue_Off (void)

  {       
       GPIO_PORTF_DATA_R &= ~0X04;   
  }
  
  
  
  
  static void Handler(void *pvParameters)
{ while(1){
  while (locker() == 1){
	
	
 if(BTN_PRESS1 () == 1 ||  BTN_PRESS2 () ==1 )
{ 
  
  xSemaphoreGive(xBinarySemaphore1);

  
}


if(BTN_PRESS3 () ==1 || BTN_PRESS4 () ==1 )
{
  
  xSemaphoreGive(xBinarySemaphore2);
  
}




  }
}


}
 



static void Manual(void *pvParameters)
{
while(1)
{ 
  xSemaphoreTake(xBinarySemaphore1,portMAX_DELAY);

if (BTN_PRESS1()==1 && Limit_PRESS3() ==0)
   {Red_On();}
if(BTN_PRESS1()==0 ){Red_Off();}
  if (BTN_PRESS1()==1 && Limit_PRESS3() ==1)
   {Red_Off();}
if(BTN_PRESS2()==1&& Limit_PRESS4() ==0){Blue_On();}
 if(BTN_PRESS2()==0){Blue_Off();}
 if(BTN_PRESS2()==1&& Limit_PRESS4() ==1){Blue_Off();} 
 
 
  
}
}



static void Automatic(void *pvParameters)
{
while(1)
{ 
  xSemaphoreTake(xBinarySemaphore2,portMAX_DELAY);

if(BTN_PRESS3()==1)
{
  while(Limit_PRESS3() ==0 && BTN_PRESS2()==0 && BTN_PRESS4()==0 && BTN_PRESS1()==0){Red_On();}
  for(int i =0; i<1600000/3;i++){Red_Off();}
   
  }
if (BTN_PRESS3()==0){Red_Off();}



if(BTN_PRESS4()==1)
{
  while(Limit_PRESS4() ==0 && BTN_PRESS1()==0 && BTN_PRESS3()==0 &&  BTN_PRESS2()==0){Blue_On();}
  for(int i =0; i<1600000/3;i++){Blue_Off();}
   
  }
if (BTN_PRESS4()==0){Blue_Off();}   

 }
 
}




  
  
  int main(){
Init_portF();
Init_portB();
  xBinarySemaphore1 = xSemaphoreCreateBinary();
  xBinarySemaphore2 = xSemaphoreCreateBinary();
xTaskCreate(Handler, "Handler",100,NULL,2,NULL);
xTaskCreate(Manual, "Manual",240,NULL,3,NULL);
xTaskCreate(Automatic, "Automatic",240,NULL,3,NULL);
vTaskStartScheduler();


}
the other code 
#include <stdint.h>

#include <string.h>

#include <FreeRTOS.h>

#include <task.h>

#include <queue.h>

#include <semphr.h>

void PortF_Init(void);

xSemaphoreHandle xMutex;

xSemaphoreHandle xBinarySemaphore1;

xSemaphoreHandle xBinarySemaphore2;

xSemaphoreHandle xBinarySemaphore3;

xSemaphoreHandle xBinarySemaphore4;

xSemaphoreHandle xBinarySemaphore5;

xSemaphoreHandle xBinarySemaphore6;

xSemaphoreHandle xBinarySemaphoreA;

xSemaphoreHandle xBinarySemaphoreB;

xSemaphoreHandle xBinarySemaph_safe

xSemaphoreHandle xBinarySemaph_jam;

xQueuHandle xmuteQueu

void Init_portF (void)

{

  

  SYSCTL->RCGCGPIO |= 0x00000020;    // 1) F clock

  GPIOF->LOCK = 0x4C4F434B;  				 // 2) unlock PortF PF0  

  GPIOF->CR = 0x1F;          				 // allow changes to PF4-0       

  GPIOF->AMSEL= 0x00;       				 // 3) disable analog function

  GPIOF->PCTL = 0x00000000;  				 // 4) GPIO clear bit PCTL  

  GPIOF->DIR = 0x0E;         				 // 5) PF4,PF0 input, PF3,PF2,PF1 output   

  GPIOF->AFSEL = 0x00;      				 // 6) no alternate function

  GPIOF->PUR = 0x11;       				   // enable pullup resistors on PF4,PF0       

  GPIOF->DEN = 0x1F;       				   // 7) enable digital pins PF4-PF0

	GPIOF->DATA = 0x00;

   

}

void Init_portB (void)

{

  

  SYSCTL->RCGCGPIO |= 0x00000002;    // 1) B clock

 

  GPIOB->CR = 0xFF;          				 // allow changes to PB4-0       

   GPIOB->DIR = 0x00;         				 // 5) PB4,PB0 input, PB3,PB2,PB1 input

  GPIOB->AFSEL = 0x00;      				 // 6) no alternate function

  GPIOB->PUR = 0xFF;       				   // enable pullup resistors on PB4-->PB0       

  GPIOB->DEN = 0xFF;       				   // 7) enable digital pins PB4-PB0

	   

}

int Btn_PRESS1 (void)

{

  if ((GPIOF->DATA & 0x10)== 0)

    return 1;

  else 

    return 0;

}

int Btn_PRESS3 (void)

{

  if ((GPIOB->DATA & 0x10)== 0)

    return 1;

  else 

    return 0;

}

void Red_On (void)

  {     

       GPIOF->DATA |= 0X02;

  }

  

 

void Red_Off (void)

  {       

       GPIOF->DATA &= ~0X02;   

  }

int Btn_PRESS2 (void)

{

  if ((GPIOF->DATA & 0x01)== 0)

    return 1;

  else 

    return 0;

}

int Btn_PRESS4 (void)

{

  if ((GPIOB->DATA & 0x01)== 0)

    return 1;

  else 

    return 0;

}

void Blue_On (void)

  {     

       GPIOF->DATA |= 0X04;

  }

  

 

void Blue_Off (void)

  {       

       GPIOF->DATA &= ~0X04;   

  }

static void safe( void *pvParameters )

{

xSemaphoreTake(xBinarySemaph_safe,0);

while(1)

{

xSemaphoreTake(xBinarySemaph_safe,portMAX_DELAY);

if(lock==0 && jamlimit==0)

{

// vTaskDelay( 250 / portTICK_RATE_MS );

xSemaphoreGive( xMutex );

}

  

  

}

static void jam (void *pvParameters)

{

xSemaphoreTake (xBinarySemaph_jam, 0);

while(1)

{

		xSemaphoreTake (xBinarySemaph_jam, portMAX_DELAY);

      Red_Off();

      Blue_Off();

	for(int i =0; i < 16000000*2;i++){int c = 0; c++;} 	 

 

int flag = 1;

		xQueueSend(xmuteQueue,&flag,xDelay);

}

static void Man_Up_pass(void *pvParameters)

{

xSemaphoreTake( xBinarySemaphore1, 0 );

while(1)

{ 

  xSemaphoreTake(xBinarySemaphore1,portMAX_DELAY);

	

   while(Btn_PRESS1()==1 && lock==0)

   {

    Red_On ();

   }

	 portBASE_TYPE xHigherPriorityTaskWoken = pdFALSE;

    xSemaphoreGiveFromISR(xBinarySemaphore3, &xHigherPriorityTaskWoken);

 portEND_SWITCHING_ISR( xHigherPriorityTaskWoken );

}

}

static void Man_Down_pass(void *pvParameters)

{

while(1)

{ 

  xSemaphoreTake(xBinarySemaphore2,portMAX_DELAY);

   while(Btn_PRESS2()==1 && lock==0)

   {

Blue_On();

   }

	 portBASE_TYPE xHigherPriorityTaskWoken = pdFALSE;

      xSemaphoreGiveFromISR(xBinarySemaphore3, &xHigherPriorityTaskWoken);

 portEND_SWITCHING_ISR( xHigherPriorityTaskWoken );

  

}

}

static void pass_Handler(void *pvParameters)

{ 

xSemaphoreTake( xMutex, portMAX_DELAY );

while(1){

	

 if(Btn_PRESS1() == 1 )

{ 

  

  portBASE_TYPE xHigherPriorityTaskWoken = pdFALSE;

xSemaphoreGiveFromISR(xBinarySemaphore1, &xHigherPriorityTaskWoken);

   portEND_SWITCHING_ISR( xHigherPriorityTaskWoken );

}

if(Btn_PRESS2() == 1 )

{

	portBASE_TYPE xHigherPriorityTaskWoken = pdFALSE;

 xSemaphoreGiveFromISR(xBinarySemaphore2, &xHigherPriorityTaskWoken);

   portEND_SWITCHING_ISR( xHigherPriorityTaskWoken );

  

}

 if(Btn_PRESS3() == 1 )

{ 

  

 portBASE_TYPE xHigherPriorityTaskWoken = pdFALSE;

 xSemaphoreGiveFromISR(xBinarySemaphore3, &xHigherPriorityTaskWoken);

	 portEND_SWITCHING_ISR( xHigherPriorityTaskWoken );

}

if(Btn_PRESS4() == 1 )

{

  

portBASE_TYPE xHigherPriorityTaskWoken = pdFALSE;

 xSemaphoreGiveFromISR(xBinarySemaphore3, &xHigherPriorityTaskWoken);

	 portEND_SWITCHING_ISR( xHigherPriorityTaskWoken );

}

xSemaphoreGive( xMutex );

}

}

static void manccw_drive( void *pvParameters)

{

xSemaphoreTake( xBinarySemaphore4, 0 );

while(1)

{ 

  xSemaphoreTake(xBinarySemaphore4,portMAX_DELAY);

	

   while(Btn_PRESS6()==1 && lock=0)

   {

    Red_On ();

   }

	 portBASE_TYPE xHigherPriorityTaskWoken = pdFALSE;

    xSemaphoreGiveFromISR(xBinarySemaphore6, &xHigherPriorityTaskWoken);

 portEND_SWITCHING_ISR( xHigherPriorityTaskWoken );

}

}

static void mancw_drive( void *pvParameters)

{

xSemaphoreTake( xBinarySemaphore5, 0 );

while(1)

{ 

  xSemaphoreTake(xBinarySemaphore5,portMAX_DELAY);

	

   while(Btn_PRESS5()==1 && lock=0)

   {

    Red_On ();

   }

	 portBASE_TYPE xHigherPriorityTaskWoken = pdFALSE;

    xSemaphoreGiveFromISR(xBinarySemaphore6, &xHigherPriorityTaskWoken);

 portEND_SWITCHING_ISR( xHigherPriorityTaskWoken );

}

}

}

static void Auto_pass_down( void *pvParameters)

{

xSemaphoreTake( xBinarySemaphore3, 0 );

while(1)

{

	

  //semaphore 3 take

 xSemaphoreTake(xBinarySemaphore3, portMAX_DELAY);

if( Btn_PRESS3()==1 )

{

     int flg = 0;

	if(xQueueReceive(xmuteQueue, &flg,0) == pdTRUE)

{

 while(Btn_PRESS1()==0 && Btn_PRESS2()==0 && Btn_PRESS4()==0 &&Btn_PRESS5()==0 && Btn_PRESS6()==0 && Btn_PRESS7()==0 && Btn_PRESS8()==0  && lock==0 && limit1==0)

 {

   Red_On ();

 }

}

}

  }

static void Auto_pass_up( void *pvParameters)

{

xSemaphoreTake( xBinarySemaphoreA, 0 );

while(1)

{

 xSemaphoreTake(xBinarySemaphoreA, portMAX_DELAY);

if(Btn_PRESS4()==1)

{

if(limit=1)

{

      int flg = 0;

	if(xQueueReceive(xmuteQueue, &flg,0) == pdTRUE)

  while(Btn_PRESS1()==0 && Btn_PRESS2()==0 && Btn_PRESS3()==0 && Btn_PRESS5()==0 && Btn_PRESS6()==0 && Btn_PRESS7()==0 && Btn_PRESS8()==0 && lock==0 && limit2==0)

  {

Blue_On ();

}

}

}

}

static void Auto_driver_up( void *pvParameters)

{

	 xSemaphoreTake(xBinarySemaphore6, 0);

while(1)

{

	

  //semaphore 6 take

 xSemaphoreTake(xBinarySemaphore6, portMAX_DELAY);

if( Btn_PRESS7()==1)

{

if(limit=1)

{

int flg = 0;

	if(xQueueReceive(xmuteQueue, &flg,0) == pdTRUE)

{

 if(Btn_PRESS5()==0 && Btn_PRESS6()==0 && Btn_PRESS8()==0 && lock==0 && limit1==0 )

 {

   Red_On ();

 }

}

}

 }

 

}

}

static void Auto_driver_down( void *pvParameters)

{

	 xSemaphoreTake(xBinarySemaphoreB, 0);

while(1)

{

	

 xSemaphoreTake(xBinarySemaphoreB, portMAX_DELAY);

if(Btn_PRESS8()==1)

{

  if(Btn_PRESS5()==0 && Btn_PRESS6()==0 && Btn_PRESS7==0 && lock==0 && limit2==0)

  {

Blue_On ();

  }

}

}

}

static void driver_control( void *pvParameters)

{

xSemaphoreTake( xMutex, portMAX_DELAY );

while(1)

{

 if(Btn_PRESS5() == 1 )

{ 

  

  portBASE_TYPE xHigherPriorityTaskWoken = pdFALSE;

xSemaphoreGiveFromISR(xBinarySemaphore4, &xHigherPriorityTaskWoken);

   portEND_SWITCHING_ISR( xHigherPriorityTaskWoken );

}

if(Btn_PRESS6() == 1 )

{

	portBASE_TYPE xHigherPriorityTaskWoken = pdFALSE;

 xSemaphoreGiveFromISR(xBinarySemaphore5, &xHigherPriorityTaskWoken);

   portEND_SWITCHING_ISR( xHigherPriorityTaskWoken );

  

}

 if(Btn_PRESS7() == 1 )

{ 

  

 portBASE_TYPE xHigherPriorityTaskWoken = pdFALSE;

 xSemaphoreGiveFromISR(xBinarySemaphore6, &xHigherPriorityTaskWoken);

	 portEND_SWITCHING_ISR( xHigherPriorityTaskWoken );

}

}

if(Btn_PRESS8() == 1 )

{

  

portBASE_TYPE xHigherPriorityTaskWoken = pdFALSE;

 xSemaphoreGiveFromISR(xBinarySemaphoreB, &xHigherPriorityTaskWoken);

	 portEND_SWITCHING_ISR( xHigherPriorityTaskWoken );

}

xSemaphore give( xMutex, portMAX_DELAY );

}

}

void ISR( void )

{

portBASE_TYPE xHigherPriorityTaskWoken = pdFALSE;

    if(jamlimit==1)

{

    xSemaphoreGiveFromISR( xBinarySemaph_jam, &xHigherPriorityTaskWoken );

    

    mainCLEAR_INTERRUPT();

   

    portEND_SWITCHING_ISR( xHigherPriorityTaskWoken );

}

if(lock==1)

{

xSemaphoreGiveFromISR( xBinarySemaph_jam, &xHigherPriorityTaskWoken );

    

    mainCLEAR_INTERRUPT();

   

    portEND_SWITCHING_ISR( xHigherPriorityTaskWoken );

}

}

int main()

{

  Init_portF();

  xBinarySemaphore1 = xSemaphoreCreateBinary();

  xBinarySemaphore2 = xSemaphoreCreateBinary();

  xBinarySemaphore3 = xSemaphoreCreateBinary();

 xBinarySemaphore4 = xSemaphoreCreateBinary();

 xBinarySemaphore5 = xSemaphoreCreateBinary();

 xBinarySemaphore6 = xSemaphoreCreateBinary();

 xBinarySemaphoreA = xSemaphoreCreateBinary();

 xBinarySemaphoreB = xSemaphoreCreateBinary();

 xBinarySemaph_jam = xSemaphoreCreateBinary();

xBinarySemaph_safe = xSemaphoreCreateBinary();

	xMutex = xSemaphoreCreateMutex();

 

	

	while(xMutex!=NULL &&  xmuteQueue!=NULL  xBinarySemaphore1!=NULL && xBinarySemaphore2!=NULL && xBinarySemaphore3!=NULL && xBinarySemaphore4!=NULL && xBinarySemaphore5!=NULL&& xBinarySemaphore6!=NULL && xBinarySemaphoreA!=NULL && xBinarySemaphoreB!=NULL && xBinarySemaph_jam!=NULL)

{

xTaskCreate(safe, "lock",100,NULL,4,NULL);

xTaskCreate(Auto_driver_up, "",100,NULL,2,NULL);

xTaskCreate(Auto_driver_down, "",100,NULL,2,NULL);

xTaskCreate(Auto_pass_up, "",100,NULL,1,NULL);

xTaskCreate(Auto_pass_down, "",100,NULL,1,NULL);

xTaskCreate(driver_control, "",100,NULL,3,NULL);

xTaskCreate(man_down_pass, "",100,NULL,1,NULL);

xTaskCreate(man_up_pass, "",100,NULL,1,NULL);

xTaskCreate(man_ccw_driver, "",100,NULL,2,NULL);

xTaskCreate(man_cw_driver, "",100,NULL,2,NULL);

xTaskCreate(pass_handler, "",100,NULL,1,NULL);

xTaskCreate(jam, "jam",100,NULL,3,NULL);

xmuteQueue = xQueueCreate( 1, sizeof(int));

	vTaskStartScheduler();

for(;;){}

}

}
