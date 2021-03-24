#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <unistd.h>
#include <string.h>
#include <bcm2835.h>

#define year    0x01
#define month   0x01
#define day     0x01
#define weekday 0x01
#define hour    0x00
#define minute  0x00
#define second  0x00

#define unusedbits_1    0x7F
#define unusedbits_2    0X3F
#define unusedbits_3    0X1F
#define unusedbits_5    0X07

#define	sec	 0
#define	MAX	 30
#define	min	 1
#define	hr 2
#define	dayw 3
#define	daym 4
#define	mon  5
#define	yea  6
#define	len	 8
#define	Temp 77
#define	Address_Byte	1
#define	Time_Bytes		7
#define	wait		5000
#define	Baud_Rate		10000
#define	RTC	0x68

typedef struct
{
	char Record1[75];
    char Record2[75];
    char Record3[75];
    unsigned int times;
}Records;

int main(int argc, char **argv)
{
	char buf[]={second, second, minute, hour, weekday, day, month, year};
  
	char *str[]  ={"Mon", "Tues", "Wed", "Thur", "Fri", "Sat","Sun"};
  
	Records rec = {"No information","No information","No information",1};
	short cycles=sec;
  
    if (!bcm2835_init())
        return 1;
        
    bcm2835_i2c_begin();
    bcm2835_i2c_setSlaveAddress(RTC);
    bcm2835_i2c_set_baudrate(Baud_Rate);
    printf("START \n");
    
    bcm2835_i2c_write(buf, len);
    while(1)
    {
		buf[sec] = second;
		bcm2835_i2c_setSlaveAddress(Temp);
		char data= bcm2835_i2c_read(buf, min);
		printf("Temperature: %d \n", buf[sec]);
		int dcon = (int) buf[sec];
    
		bcm2835_i2c_setSlaveAddress(RTC);
		buf[sec] = second;
		bcm2835_i2c_write_read_rs(buf, Address_Byte, buf, Time_Bytes);
		buf[sec] = buf[sec] & unusedbits_1;
		buf[min] = buf[min] & unusedbits_1;
		buf[hr] = buf[hr] & unusedbits_2;
		buf[dayw] = buf[dayw] & unusedbits_5;
		buf[daym] = buf[daym] & unusedbits_2;
		buf[mon] = buf[mon] & unusedbits_3;
    
		if (dcon>=MAX || buf[sec] == 0x00 || buf[sec] == 0x10 || buf[sec] == 0x20 || buf[sec] == 0x30 || buf[sec] == 0x40 || buf[sec] == 0x50){

			strcpy(rec.Record3,rec.Record2);
			strcpy(rec.Record2,rec.Record1);
			snprintf(rec.Record1,75,"Record %i: %02x/%02x/%02x %s %02x:%02x:%02x", rec.times, buf[yea], buf[mon], buf[daym], str[(unsigned char)buf[dayw]-1], buf[hr], buf[min], buf[sec]);
			rec.times = rec.times + min;
      
			FILE *f = fopen("Datalog.txt", "w");
			printf("%s \n",rec.Record1);
			printf("%s \n",rec.Record2);
			printf("%s \n",rec.Record3);
			fprintf(f,"%s \n",rec.Record1);
			fprintf(f,"%s \n",rec.Record2);
			fprintf(f,"%s \n",rec.Record3);
      
			fclose(f);
		}
    
		bcm2835_delay(wait);
    }

    bcm2835_i2c_end();
    bcm2835_close();
    return 0;
}
