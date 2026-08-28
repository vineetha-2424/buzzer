#include <reg51.h>

#define PASSWORD "1234"

sbit RS = P3^0;
sbit EN = P3^1;
sbit LOCK = P3^2;
sbit BUZZER = P3^3;

void delay(unsigned int ms)
{
    unsigned int i, j;

    for(i = 0; i < ms; i++)
        for(j = 0; j < 1275; j++);
}

void lcd_cmd(unsigned char cmd)
{
    P2 = cmd;
    RS = 0;
    EN = 1;
    delay(2);
    EN = 0;
}

void lcd_data(unsigned char data1)
{
    P2 = data1;
    RS = 1;
    EN = 1;
    delay(2);
    EN = 0;
}

void lcd_init()
{
    lcd_cmd(0x38);   // 8-bit, 2-line LCD
    lcd_cmd(0x0C);   // Display ON, cursor OFF
    lcd_cmd(0x01);   // Clear display
    lcd_cmd(0x06);   // Increment cursor
    lcd_cmd(0x80);   // First line
}

void lcd_string(char *str)
{
    while(*str)
    {
        lcd_data(*str++);
    }
}

char keypad()
{
    unsigned char row, col;

    char keys[4][4] =
    {
        {'1','2','3','A'},
        {'4','5','6','B'},
        {'7','8','9','C'},
        {'*','0','#','D'}
    };

    while(1)
    {
        for(row = 0; row < 4; row++)
        {
            P1 = 0xFF;
            P1 &= ~(1 << row);

            if((P1 & 0xF0) != 0xF0)
            {
                delay(20);

                if((P1 & 0xF0) != 0xF0)
                {
                    if((P1 & 0x10) == 0)
                        col = 0;
                    else if((P1 & 0x20) == 0)
                        col = 1;
                    else if((P1 & 0x40) == 0)
                        col = 2;
                    else
                        col = 3;

                    while((P1 & 0xF0) != 0xF0);

                    return keys[row][col];
                }
            }
        }
    }
}

void main()
{
    char password[5];
    char key;
    unsigned char i;
    bit correct;

    LOCK = 1;       // Door locked
    BUZZER = 0;

    lcd_init();

    while(1)
    {
        lcd_cmd(0x01);
        lcd_string("Enter Password:");
        lcd_cmd(0xC0);

        for(i = 0; i < 4; i++)
        {
            key = keypad();

            if(key == '#')
                break;

            password[i] = key;
            lcd_data('*');
        }

        password[4] = '\0';

        correct = 1;

        for(i = 0; i < 4; i++)
        {
            if(password[i] != PASSWORD[i])
            {
                correct = 0;
                break;
            }
        }

        lcd_cmd(0x01);

        if(correct)
        {
            lcd_string("Access Granted");

            LOCK = 0;       // Unlock door
            delay(5000);

            LOCK = 1;       // Lock door again

            lcd_cmd(0x01);
            lcd_string("Door Locked");
            delay(1000);
        }
        else
        {
            lcd_string("Wrong Password");

            BUZZER = 1;
            delay(1000);
            BUZZER = 0;

            lcd_cmd(0x01);
            lcd_string("Access Denied");
            delay(1000);
        }
    }
}# buzzer
