# A-Design-of-Intelligent-Trajectory-Car-Based-on-aduino
A Design of Intelligent Trajectory Car Based on aduino ->>>how to make a car to Automatic tracking and balancing


#include "Wire.h"
#include "I2Cdev.h"
#include "MPU6050.h"

/*#define S 90  //占空比大小*/

volatile float S = 90;

int IN1=12;
int IN2=13;//前左轮定义
int IN3=9;
int IN4=8;//前右轮定义

int IN5=A0; //后右轮
int IN6=A1;

int IN7=A2;
int IN8=A3;//后左轮

int XJ1=2;
int XJ2=3;
int XJ3=4;
int val1=0;//定义LED 接口
int val2=0;//定义LED 接口
int val3=0;//定义LED 接口

int EN1=6;
int EN2=5;  //前轮使能
int EN3=10;
int EN4=11; //后轮使能

int beep=7; //beep 蜂鸣器

MPU6050 accelgyro;

unsigned long now, lastTime = 0;
float dt;                                   //微分时间

int16_t ax, ay, az, gx, gy, gz;             //加速度计陀螺仪原始数据
float aax=0, aay=0,aaz=0, agx=0, agy=0, agz=0;    //角度变量
long axo = 0, ayo = 0, azo = 0;             //加速度计偏移量
long gxo = 0, gyo = 0, gzo = 0;             //陀螺仪偏移量

//float avg

float pi = 3.1415926;
float AcceRatio = 16384.0;                  //加速度计比例系数
float GyroRatio = 131.0;                    //陀螺仪比例系数

uint8_t n_sample = 8;                       //加速度计滤波算法采样个数
float aaxs[8] = {0}, aays[8] = {0}, aazs[8] = {0};         //x,y轴采样队列
long aax_sum, aay_sum,aaz_sum;                      //x,y轴采样和

float a_x[10]={0}, a_y[10]={0},a_z[10]={0} ,g_x[10]={0} ,g_y[10]={0},g_z[10]={0}; //加速度计协方差计算队列
float Px=1, Rx, Kx, Sx, Vx, Qx;             //x轴卡尔曼变量
float Py=1, Ry, Ky, Sy, Vy, Qy;             //y轴卡尔曼变量
float Pz=1, Rz, Kz, Sz, Vz, Qz;             //z轴卡尔曼变量

void setup() {
  // put your setup code here, to run once:
pinMode(IN1,OUTPUT);
pinMode(IN2,OUTPUT);
pinMode(IN3,OUTPUT);
pinMode(IN4,OUTPUT);

pinMode(IN5,OUTPUT);
pinMode(IN6,OUTPUT);
pinMode(IN7,OUTPUT);
pinMode(IN8,OUTPUT);

pinMode(XJ1,INPUT);//定义避障传感器为输出接口
pinMode(XJ2,INPUT);//定义避障传感器为输出接口
pinMode(XJ3,INPUT);//定义避障传感器为输出接口

pinMode(EN1,OUTPUT);//设置数字端口5  6为输出模式
pinMode(EN2,OUTPUT);
pinMode(EN3,OUTPUT);
pinMode(EN4,OUTPUT);

pinMode(beep,OUTPUT);

    Wire.begin();
 //   Serial.begin(115200);

    accelgyro.initialize();                 //初始化

    unsigned short times = 300;             //采样次数
    for(int i=0;i<times;i++)
    {
        accelgyro.getMotion6(&ax, &ay, &az, &gx, &gy, &gz); //读取六轴原始数值
        axo += ax; ayo += ay; azo += az;      //采样和
        gxo += gx; gyo += gy; gzo += gz;       
    }
    
    axo /= times; ayo /= times; azo /= times; //计算加速度计偏移
    gxo /= times; gyo /= times; gzo /= times; //计算陀螺仪偏移

}
void MotorZ()//正转
{
    analogWrite(EN1,S);
    analogWrite(EN2,S);
    
    analogWrite(EN3,S);
    analogWrite(EN4,S);
    
    digitalWrite(IN1,LOW);
    digitalWrite(IN2,HIGH);
    digitalWrite(IN3,LOW);    
    digitalWrite(IN4,HIGH);
    
    digitalWrite(A0,HIGH);
    digitalWrite(A1,LOW); 
    digitalWrite(A2,HIGH);
    digitalWrite(A3,LOW);
        
}
void MotorF()//反转
{
    analogWrite(EN1,S);
    analogWrite(EN2,S);
    
    digitalWrite(IN1,HIGH);
    digitalWrite(IN2,LOW);
    digitalWrite(IN3,HIGH);
    digitalWrite(IN4,LOW);
    
    analogWrite(EN3,S);
    analogWrite(EN4,S);

    digitalWrite(A0,LOW);
    digitalWrite(A1,HIGH);
    digitalWrite(A2,LOW);
    digitalWrite(A3,HIGH);
      
}

void MotorRIGHT()//正双右转
{    
    analogWrite(EN1,S);
    analogWrite(EN2,S);
    
    digitalWrite(IN1,LOW);
    digitalWrite(IN2,HIGH);
    digitalWrite(IN3,HIGH);
    digitalWrite(IN4,LOW);

    analogWrite(EN3,S);
    analogWrite(EN4,S);

    digitalWrite(A0,HIGH);
    digitalWrite(A1,LOW);
    digitalWrite(A2,HIGH);
    digitalWrite(A3,LOW);
    
}

void MotorRIGHTF()//反双右转
{    
    analogWrite(EN1,S);
    analogWrite(EN2,S);
    
    analogWrite(EN3,S);
    analogWrite(EN4,S);
    
    digitalWrite(IN1,LOW);
    digitalWrite(IN2,HIGH);
    digitalWrite(IN3,HIGH);
    digitalWrite(IN4,LOW);
    
    digitalWrite(A0,LOW);
    digitalWrite(A1,HIGH);
    digitalWrite(A2,LOW);
    digitalWrite(A3,HIGH);
    
}

void MotorLEFT()//正双左转
{
    analogWrite(EN1,S);
    analogWrite(EN2,S);
    
    digitalWrite(IN1,HIGH);
    digitalWrite(IN2,LOW);
    digitalWrite(IN3,LOW);
    digitalWrite(IN4,HIGH);
    
    analogWrite(EN3,S);
    analogWrite(EN4,S);

    digitalWrite(A0,HIGH);
    digitalWrite(A1,LOW);
    digitalWrite(A2,HIGH);
    digitalWrite(A3,LOW);    
}

void MotorLEFTF()//反双左转
{
    analogWrite(EN1,S);
    analogWrite(EN2,S);
    
    analogWrite(EN3,S);
    analogWrite(EN4,S);
    
    digitalWrite(IN1,HIGH);
    digitalWrite(IN2,LOW);
    digitalWrite(IN3,LOW);
    digitalWrite(IN4,HIGH);
    
    digitalWrite(A0,LOW);
    digitalWrite(A1,HIGH);
    digitalWrite(A2,LOW);
    digitalWrite(A3,HIGH);
}

void ReadData(){
    unsigned long now = millis();             //当前时间(ms)
    dt = (now - lastTime) / 1000.0;           //微分时间(s)
    lastTime = now;                           //上一次采样时间(ms)

    accelgyro.getMotion6(&ax, &ay, &az, &gx, &gy, &gz); //读取六轴原始数值

    float accx = ax / AcceRatio;              //x轴加速度
    float accy = ay / AcceRatio;              //y轴加速度
    float accz = az / AcceRatio;              //z轴加速度

    aax = atan(accy / accz) * (-180) / pi;    //y轴对于z轴的夹角
    aay = atan(accx / accz) * 180 / pi;       //x轴对于z轴的夹角
    aaz = atan(accz / accy) * 180 / pi;       //z轴对于y轴的夹角

    aax_sum = 0;                              // 对于加速度计原始数据的滑动加权滤波算法
    aay_sum = 0;
    aaz_sum = 0;
  
    for(int i=1;i<n_sample;i++)
    {
        aaxs[i-1] = aaxs[i];
        aax_sum += aaxs[i] * i;
        aays[i-1] = aays[i];
        aay_sum += aays[i] * i;
        aazs[i-1] = aazs[i];
        aaz_sum += aazs[i] * i;
    
    }
    
    aaxs[n_sample-1] = aax;
    aax_sum += aax * n_sample;
    aax = (aax_sum / (11*n_sample/2.0)) * 9 / 7.0; //角度调幅至0-90°
    aays[n_sample-1] = aay;                        //此处应用实验法取得合适的系数
    aay_sum += aay * n_sample;                     //本例系数为9/7
    aay = (aay_sum / (11*n_sample/2.0)) * 9 / 7.0;
    aazs[n_sample-1] = aaz; 
    aaz_sum += aaz * n_sample;
    aaz = (aaz_sum / (11*n_sample/2.0)) * 9 / 7.0;

    float gyrox = - (gx-gxo) / GyroRatio * dt; //x轴角速度
    float gyroy = - (gy-gyo) / GyroRatio * dt; //y轴角速度
    float gyroz = - (gz-gzo) / GyroRatio * dt; //z轴角速度
    agx += gyrox;                             //x轴角速度积分
    agy += gyroy;                             //y轴角速度积分
    agz += gyroz;
    
    /* kalman start */
    Sx = 0; Rx = 0;
    Sy = 0; Ry = 0;
    Sz = 0; Rz = 0;
    
    for(int i=1;i<10;i++)
    {                                           //测量值平均值运算
        a_x[i-1] = a_x[i];                      //即加速度平均值
        Sx += a_x[i];
        a_y[i-1] = a_y[i];
        Sy += a_y[i];
        a_z[i-1] = a_z[i];
        Sz += a_z[i];
    
    }
    
    a_x[9] = aax;
    Sx += aax;
    Sx /= 10;                                 //x轴加速度平均值
    a_y[9] = aay;
    Sy += aay;
    Sy /= 10;                                 //y轴加速度平均值
    a_z[9] = aaz;
    Sz += aaz;
    Sz /= 10;

    for(int i=0;i<10;i++)
    {
        Rx += sq(a_x[i] - Sx);
        Ry += sq(a_y[i] - Sy);
        Rz += sq(a_z[i] - Sz);
    
    }
    
    Rx = Rx / 9;                              //得到方差
    Ry = Ry / 9;                        
    Rz = Rz / 9;
  
    Px = Px + 0.0025;                         // 0.0025在下面有说明...
    Kx = Px / (Px + Rx);                      //计算卡尔曼增益
    agx = agx + Kx * (aax - agx);             //陀螺仪角度与加速度计速度叠加
    Px = (1 - Kx) * Px;                       //更新p值

    Py = Py + 0.0025;
    Ky = Py / (Py + Ry);
    agy = agy + Ky * (aay - agy); 
    Py = (1 - Ky) * Py;
  
    Pz = Pz + 0.0025;
    Kz = Pz / (Pz + Rz);
    agz = agz + Kz * (aaz - agz); 
    Pz = (1 - Kz) * Pz;
    /* kalman end */ 
}

void Stop(){
    analogWrite(EN1,S);
    analogWrite(EN2,S);
    
    digitalWrite(IN1,LOW);
    digitalWrite(IN2,LOW);
    digitalWrite(IN3,LOW);
    digitalWrite(IN4,LOW);
    
    analogWrite(EN3,S);
    analogWrite(EN4,S);

    digitalWrite(A0,LOW);
    digitalWrite(A1,LOW);
    digitalWrite(A2,LOW);
    digitalWrite(A3,LOW);   
}

void beepon(){
  digitalWrite(beep,HIGH); //鸣叫提示
  delay(5000); 
  digitalWrite(beep,LOW);
}

void toph(){
  if( agy > 2.50 && agy < 10){        //正姿态
        if(val2 == 1){   //中感应
          if(val1 == 1 && val3 == 0){  //左中 右转
            MotorRIGHT();
            delay(100);
            Stop();
            delay(500);
          }
          else if(val1 == 0 && val3 == 1){    //右中 左转
            MotorLEFT();
            delay(100);
            Stop();
            delay(500);         
          }
          else{
            MotorZ();   //前进
            delay(100);
            Stop();
            delay(500);
          }
        }
        if(val1==1&&val2==0&&val3==0){  //左  右转
            MotorRIGHT();
            delay(100);
            Stop();
            delay(500);
        }
        if(val1==0&&val2==0&&val3==1){  //右  左转
            MotorLEFT();
            delay(100);
            Stop();
            delay(500);
        }
        if(val1==0&&val2==0&val3==0){   // 空 停止
            Stop();     
        }  
      }
      else if( agy > -6.5 && agy < 0.00){  //负姿态
          if(val2 == 1){           
              MotorF();
              delay(100);
              Stop();
              delay(500);          
          }
          if(val1==1&&val2==0&&val3==0){   //左 （负向 右）
              MotorRIGHTF();
              delay(100);
              Stop();
              delay(500);
          }
          if(val1==0&&val2==0&&val3==1){   // 右 （左）
              MotorLEFTF();
              delay(50);
              Stop();
              delay(500);
          }
          if(val1==0&&val2==0&val3==0){  //停
              Stop();     
          }  
        } 
}

void gotozd(){     
      if(val2 == 1){   //中感应
          if(val1 == 1 && val3 == 0){  //左中 右转
            MotorRIGHT();
            delay(60);
            Stop();
            delay(500);
          }
          else if(val1 == 0 && val3 == 1){    //右中 左转
            MotorLEFT();
            delay(60);
            Stop();
            delay(500);         
          }
          else{
            MotorZ();   //前进
            delay(60);
            Stop();
            delay(500);
          }
        }
        if(val1==1&&val2==0&&val3==0){  //左  右转
            MotorRIGHT();
            delay(60);
            Stop();
            delay(500);
        }
        if(val1==0&&val2==0&&val3==1){  //右  左转
            MotorLEFT();
            delay(60);
            Stop();
            delay(500);
        }            
}


void gotoB(){
          if(val2 == 1){           
              MotorF();
              delay(100);
              Stop();
              delay(500);          
          }
          if(val1==1&&val2==0&&val3==0){   //左 （负向 右）
              MotorRIGHTF();
              delay(100);
              Stop();
              delay(500);
          }
          if(val1==0&&val2==0&&val3==1){   // 右 （左）
              MotorLEFTF();
              delay(50);
              Stop();
              delay(500);
          }       
}


void read(){
  ReadData(); 
  delay(100);
}

void Speedback(){
  if(aay>0){
      if(S > 65){
      S = 15 * aay;
      }
      if( S > 0 && S <= 65){
        S = 60 + 10 * aay;
      }
    }
  if(aay<0){
     if(S > 65){
      S = 15 * (-aay);
      }
      if( S > 0 && S <= 65){
        S = 60 + 10 * (-aay);
      }
    }  
}

void Speedbacktwo(){
  if(aay>0){
      if(S > 70){
      S = 15 * aay;
      }
      if( S > 0 && S <= 70){
        S = 70 + 10 * aay;
      }
    }
  if(aay<0){
     if(S > 70){
      S = 15 * (-aay);
      }
      if( S > 0 && S <= 70){
        S = 70 + 10 * (-aay);
      }
    }  
}

void loop(){
  read();
  val1=digitalRead(XJ1); /*正向左传感器*/
  val2=digitalRead(XJ2); 
  val3=digitalRead(XJ3); 
  Speedback();
  if (agy >= 1.40 && agy <= 2.50) {
      Stop();    
      delay(100); 
      if (agy >= 1.40 && agy <= 2.50){
        Stop(); 
        S = 60;       
              if(val2 == 1&&val1==0&&val3==0){                 
                if (S =60){              
                 Stop();                                   
                 beepon(); 
                    while(1){
                      read();
                      val1=digitalRead(XJ1); /*正向左传感器*/
                      val2=digitalRead(XJ2); 
                      val3=digitalRead(XJ3); 
                      Speedbacktwo();
                      gotozd();
                      if(val1==0&&val2==0&val3==0){   /*/ 空 停止*/
                      Stop();                     
                        while(1){
                          read();
                          val1=digitalRead(XJ1); /*正向左传感器*/
                          val2=digitalRead(XJ2); 
                          val3=digitalRead(XJ3); 
                          Speedbacktwo();
                          Stop();
                          delay(5000);  
                          MotorF();
                          delay(2500);
                          Stop();
                          delay(500);                         
                          gotoB();
                          if(val1==0&&val2==0&val3==0){   /*/ 空 停止*/
                          Stop();
                          break;
                          }
                        }
                        break;          
                      }   
                    }
                 }                                                                                                                                                                       
              }         
           }
       } 
    toph();  
    
}  
