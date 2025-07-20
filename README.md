# EV Route Planner
-Designed a dynamic decision support system to recommend optimal charging stations along a route, based on user-defined parameters and the weighted importance of cost vs. time objectives. 
-Incorporated temperature and road profile data into the optimization  model to reflect real-world driving conditions. 
-Used Google Earth Pro to extract and analyze elevation profiles of selected routes for energy consumption modeling. 
-Developed a statistical model to estimate real-time temperature using historical and forecast data, enhancing the accuracy of energy consumption calculations.
-Conducted extensive literature research on factors affecting energy consumption and battery health, and integrated them into the mathematical formulation. 
-Gained strong experience in building linear yet dynamic optimization models, particularly for applications in smart mobility and sustainable transportation. 

# The Example GAMS Code for the route of Ankara-Bursa
Sets
    i /1*39/
    j /1*10/;
Alias (i,ip), (j,jp);

Scalars
    Binit "BaÅŸlangÄ±Ã§ batarya seviyesi (%)" /100/
    OutsideT "DÄ±ÅŸ ortam sÄ±caklÄ±ÄŸÄ± (Â°C)" /25/
    Batterylevelf "Rota sonunda batarya seviyesi (%)" /20/
    Bmax "Batarya kapasitesi (kWh)" /70/
    OptimalT "Optimal sÄ±caklÄ±k (Â°C)" /23/
    T "Ä°stasyona toplam giriÅŸ Ã§Ä±kÄ±ÅŸ sÃ¼resi (dk)" /10/
    Alpha /0/;

* Her parÃ§a iÃ§in sÄ±caklÄ±k katsayÄ±larÄ± (OutsideTI = a(i) * OutsideT + b(i))
Parameters
    a(i) "Sıcaklık katsayısı a"/ 1 1, 2 1, 3 1, 4 1, 5 1, 6 1, 7 0.866808457, 8 0.866808457, 9 0.866808457, 10 0.866808457,
11 0.866808457, 12 0.866808457, 13 0.866808457, 14 0.866808457, 15 0.866808457, 16 0.866808457, 17 0.866808457, 18 0.866808457, 19 0.846533557, 20 0.846533557,
21 0.846533557, 22 0.846533557, 23 0.846533557, 24 0.846533557, 25 0.846533557, 26 0.846533557, 27 0.753985705, 28 0.753985705, 29 0.753985705, 30 0.753985705,
31 0.753985705, 32 0.711403942, 33 0.711403942, 34 0.711403942, 35 0.711403942, 36 0.711403942, 37 0.711403942, 38 0.711403942, 39 0.711403942/

    b(i) "Sıcaklık katsayısı b"/ 1 0, 2 0, 3 0, 4 0, 5 0, 6 0, 7 1.881143598, 8 1.881143598, 9 1.881143598, 10 1.881143598,
11 1.881143598,  12 1.881143598,  13 1.881143598,  14 1.881143598,  15 1.881143598,  16 1.881143598,  17 1.881143598,  18 1.881143598, 19 1.676462568, 20 1.676462568,
21 1.676462568,  22 1.676462568,  23 1.676462568,  24 1.676462568,  25 1.676462568,  26 1.676462568,  27 4.968677767,  28 4.968677767, 29 4.968677767, 30 4.968677767,
31 4.968677767,  32 5.459714529,  33 5.459714529,  34 5.459714529,  35 5.459714529,  36 5.459714529,  37 5.459714529,  38 5.459714529, 39 5.459714529/

    OutsideTI(i) "Her parça için dış sıcaklık etkisi (°C)"
;

* Sallama sıcaklık katsayıları
*a(i) = uniform(0.9, 1.1);
*b(i) = uniform(-2, 2);

* Sıcaklık etkisini hesapla
OutsideTI(i) = a(i) * OutsideT + b(i);


* YÃ¼kseklik farkÄ± kaynaklÄ± enerji kaybÄ± (kWh)
Parameter
m(i) "Sıcaklık katsayısı a"/ 1 -0.086979711, 2 0.368534149, 3 0.437671877 , 4 -0.314537441, 5 -0.239781906, 6 0.352903011, 7 0.458713794, 8 -0.115424427, 9 -0.273633469, 10 0.548893439,
11 -0.072874893, 12 0.444886249,   13 1.110412029,   14 0.048095811,    15 -0.073345054, 16 -0.010813694,  17 -0.136346574,   18 -0.074285375,   19 -0.007052409,   20 -0.030090278,
21 0.159317373,  22 -0.219565,     23 -0.138932457,  24 0.209817974,    25 0.104007191, 26 0.126251503,  27 -0.077576499,   28 -0.194881569,   29 0.258514982,   30 0.162323361,
31 -0.491317827, 32 -0.506833127,  33 -0.18806424,  34 -0.048896702,    35 0.789372493,  36 -0.303253587,   37 -0.41562197,   38 0.24889582, 39 0.2795569/

*Parameter m(i);
*$call =xls2gms r=A1:B60 i=C:\graduation_project\m_i_izmir_ankara.xlsx o=C:\graduation_project\izmir_ankara_miput2.inc
*$include 'C:\graduation_project\izmir_ankara_miput2.inc'

Parameter
    E0 /0/
    Consumption(i) "Her parÃ§ada toplam tÃ¼ketim (kWh)"
;

Consumption(i) $ (OutsideTI(i) >= -10 and OutsideTI(i) <= 23) = 0.01 * (-1.643 * OutsideTI(i) + 220.6) + m(i);
Consumption(i) $ (OutsideTI(i) > 23 and OutsideTI(i) <= 41) = 0.01 * (2.928 * OutsideTI(i) + 117.9) + m(i);



Table CS(i,j) Sarj istasyonu kapasitesi kWh
*$call =xls2gms r=A2:I61 i=C:\graduation_project\ankara_bursa_csu.xlsx o=C:\graduation_project\ankara_bursa_csuput.inc
$include 'C:\graduation_project\ankara_bursa_csuput.inc'
;
Parameter CSU(i,j)
;
CSU(i,j)$ ABS(OutsideTI(i)<=25)= CS(i,j)* (0.0168 * OutsideTI(i) + 0.583);
CSU(i,j)$ ABS(OutsideTI(i)>25)= CS(i,j)*(-0.027 * OutsideTI(i) + 1.585);

Table AN(i,j) AracÄ±n sarj kapasitesi kWh
*$call =xls2gms r=A2:I61 i=C:\graduation_project\ankara_bursa_an.xlsx o=C:\graduation_project\ankara_bursa_anuput.inc
$include 'C:\graduation_project\ankara_bursa_anuput.inc'
;
Parameter ANU(i,j) AracÄ±n sarj kapasitesi kWh
;
ANU(i,j)$ ABS(OutsideTI(i)<=25)= AN(i,j)* (0.0168 * OutsideTI(i) + 0.583);
ANU(i,j)$ ABS(OutsideTI(i)>25)= AN(i,j)*(-0.027 * OutsideTI(i) + 1.585);
;

Parameter Chargingcap(i,j);
Chargingcap(i,j)$(CSU(i,j)>=ANU(i,j))=ANU(i,j);
Chargingcap(i,j)$(CSU(i,j)<ANU(i,j))=CSU(i,j);

Table Chargingstation(i,j)  Ä°stasyon varsa 1 ow 0
*$call =xls2gms r=A2:I61 i=C:\graduation_project\ankara_bursa_binary.xlsx o=C:\graduation_project\ankara_bursa_binput.inc
$include 'C:\graduation_project\ankara_bursa_binput.inc'
;

Table c(i,j) i parÃ§asÄ±ndaki j istasyonunun kWh Ã¼creti
*$call =xls2gms r=A2:I61 i=C:\graduation_project\ankara_bursa_cost.xlsx o=C:\graduation_project\ankara_bursa_costput.inc
$include 'C:\graduation_project\ankara_bursa_costput.inc'
;

E0 $(Binit>0)=(Binit* Bmax)/100;
Variables
    x(i,j)    'binary const'
    E(i)    'energy at the end of road part i'
    Charge(i,j)  'charge'
    z       'toplam dakika sarp maliyeti'
    sure(i,j)
    TC
    TT
    FC
    TCT
    SC(i,j)
    f(i)
    NS
    Percentage(i)
    Qmax(i,j)
    Nt(i)
    chargingtime(i,j)
    ;
Binary Variables x,y ;

Positive Variables E, Charge(i,j), Q(i,j),Nmax(i,j),N(i,j),W(i,j),Wmax(i,j),Qmax(i,j),Nt(i),cumulative(i,j),cumulativeMax(i);


Equations
    InitialEnergy
    EnergyUpdate(i,j)
    Objective
    TotalCost
    TotalTime
    FixedCost
    TotalChargingTime
    check(i,j)
    alt
    ctime(i,j)
    StationUse(i,j)
    Echeck(i)
    maxcheckQ(i)
    maxcheckN(i)
    maxcheckW(i)
    dnm(i,j)
    NumberofStops
    kaph(i)
    PercentageofBattery(i)
    StationUse2(i,j)
    StationUse3(i,j)
    StationUse4(i)
    dnm2(i)
    maxcheckNmax(i,j)
    maxcheckWmax(i,j)
    dnm3(i)
    dnm4(i)
    NmaxLimit(i)
    cumuleg(i,j)
    cumulativeMaxDef(i,j)
    onlycharge(i,j)
    SingleCost(i,j)
    ;

Objective.. z =e= Alpha*TC+(1-Alpha)*(TT+FC)+NS;

TotalCost..TC=e=sum((i,j),c(i,j)*Charge(i,j));
TotalTime..TT=e=cumulativeMax('39');
FixedCost..FC=e=sum ((i,j),x(i,j)*T);
TotalChargingTime..TCT=e=sum ((i,j),chargingtime(i,j));
NumberofStops..NS=e=sum ((i,j),x(i,j));
SingleCost(i,j)..SC(i,j)=e=c(i,j)*Charge(i,j);

alt..sum ((i,j),Charge(i,j)) =g=sum (i,Consumption(i))-E0+((Batterylevelf* Bmax)/100) ;
Echeck(i)..f(i)=e=sum(j,Charge(i,j));
InitialEnergy.. E('1') =e= E0 - Consumption('1');
check(i,j).. Charge(i,j)=e=Charge(i,j)$(Chargingstation(i,j)=1);
EnergyUpdate(i,j)$ (ord(i) ne 1).. E(i)=e= E(i-1) - Consumption(i)+f(i);
kaph(i)..E(i)=l=Bmax;
PercentageofBattery(i)..Percentage(i)=e=(E(i)/Bmax*100);
*ctime(i,j)..sure(i,j)=e= 0.83 + (60*((N(i,j))/(Chargingcap(i,j))+((W(i,j))/7)+Q(i,j)/7))$(Chargingcap(i,j)<>0);
ctime(i,j)..sure(i,j)=e= 0.83 + (60*((N(i,j))/(Chargingcap(i,j))+((W(i,j))/(Chargingcap(i,j)*0.3))+Q(i,j)/(Chargingcap(i,j)*0.3)))$(Chargingcap(i,j)<>0);
onlycharge(i,j)..chargingtime(i,j)=e=(60*((N(i,j))/(Chargingcap(i,j))+((W(i,j))/(Chargingcap(i,j)*0.3))+Q(i,j)/(Chargingcap(i,j)*0.3)))$(Chargingcap(i,j)<>0);
cumuleg(i,j)..cumulative(i,j)=e=sum((ip,jp)$(ord(ip) < ord(i)), sure(ip,jp)) + sum(jp$(ord(jp) <= ord(j)), sure(i,jp));
cumulativeMaxDef(i,j).. cumulativeMax(i) =e= cumulative(i,'10');
StationUse(i,j).. Charge(i,j) =l= Bmax*x(i,j);
dnm(i,j)..Charge(i,j)=e=Q(i,j)+ W(i,j)+ N(i,j);
StationUse2(i,j).. Q(i,j) =l= (Bmax*0.2)*x(i,j);
StationUse3(i,j).. W(i,j) =l= (Bmax*0.2)*x(i,j);
StationUse4(i)$ (ord(i) ne 1).. sum(j,Nmax(i,j)) =l= Nt(i);
NmaxLimit(i).. Nt(i)=g=(Bmax*0.8-E(i-1)- sum(j,Q(i,j)));
maxcheckQ(i)$(ord(i) ne 1)..sum (j, Q(i,j))=g=(Bmax*0.2-E(i-1));
maxcheckN(i)$(ord(i) ne 1)..sum (j, Nmax(i,j))=g=(Bmax*0.8-E(i-1)- sum(j,Q(i,j)));
maxcheckNmax(i,j)$(ord(i) ne 1)..N(i,j)=l=Nmax(i,j);
maxcheckW(i)$(ord(i) ne 1)..sum (j, Wmax(i,j))=e=(Bmax)-E(i-1)-(sum (j, Q(i,j)))-(sum (j, Nmax(i,j)));
maxcheckWmax(i,j)$(ord(i) ne 1)..W(i,j)=l=Wmax(i,j);

dnm3(i)..sum(j,W(i,j)) =l= 0.2*Bmax*y(i);
dnm4(i)..sum (j, Nmax(i,j)-N(i,j))=l=0.6*Bmax*(1-y(i));

dnm2(i)..sum(j, x(i,j))=l=1;

Model RoadTrip /all/;
Solve RoadTrip using MIP minimizing z;

*display z.l charge.l ;

*execute_unload "C:\graduation_project\resultsbk_Q.gdx" Q.L
*execute 'gdxxrw.exe C:\graduation_project\resultsbk_Q.gdx o=C:\graduation_project\resultsbk.xlsx var=Q.L rng=Q!A1:z200'

*execute_unload "C:\graduation_project\resultsbk_nMax.gdx" Nmax.L
*execute 'gdxxrw.exe C:\graduation_project\resultsbk_nMax.gdx o=C:\graduation_project\resultsbk.xlsx var=Nmax.L rng=nmax!A1:z200'

*execute_unload "C:\graduation_project\resultsbk_N.gdx" N.L
*execute 'gdxxrw.exe C:\graduation_project\resultsbk_N.gdx o=C:\graduation_project\resultsbk.xlsx var=N.L rng=N!A1:z200'

*execute_unload "C:\graduation_project\resultsbk_wMax.gdx" Wmax.L
*execute 'gdxxrw.exe C:\graduation_project\resultsbk_wMax.gdx o=C:\graduation_project\resultsbk.xlsx var=Wmax.L rng=wmax!A1:z200'

*execute_unload "C:\graduation_project\resultsbk_W.gdx" W.L
*execute 'gdxxrw.exe C:\graduation_project\resultsbk_W.gdx o=C:\graduation_project\resultsbk.xlsx var=W.L rng=W!A1:z200'

execute_unload "C:\graduation_project\ankarabursa_E.gdx" E.L
execute 'gdxxrw.exe C:\graduation_project\ankarabursa_E.gdx o=C:\graduation_project\ankarabursa.xlsx var=E.L rng=energylevel!A1:ZZ10000'

execute_unload "C:\graduation_project\ankarabursa_Percentage.gdx" Percentage.L
execute 'gdxxrw.exe C:\graduation_project\ankarabursa_Percentage.gdx o=C:\graduation_project\ankarabursa.xlsx var=Percentage.L rng=batterypercentage!A1:ZZ10000'

execute_unload "C:\graduation_project\ankarabursa_Consumption.gdx", Consumption;
execute 'gdxxrw.exe C:\graduation_project\ankarabursa_Consumption.gdx o=C:\graduation_project\ankarabursa.xlsx par=Consumption rng=consumption!A1:ZZ10000'

execute_unload "C:\graduation_project\ankarabursa_OutsideTI.gdx", OutsideTI;
execute 'gdxxrw.exe C:\graduation_project\ankarabursa_OutsideTI.gdx o=C:\graduation_project\ankarabursa.xlsx par=OutsideTI rng=OutsideTI!A1:ZZ10000'

execute_unload "C:\graduation_project\ankarabursa_Charge.gdx" Charge.L
execute 'gdxxrw.exe C:\graduation_project\ankarabursa_Charge.gdx o=C:\graduation_project\ankarabursa.xlsx var=Charge.L rng=charge!A1:z200'

execute_unload "C:\graduation_project\ankarabursa_TC.gdx" TC.L
execute 'gdxxrw.exe C:\graduation_project\ankarabursa_TC.gdx o=C:\graduation_project\ankarabursa.xlsx var=TC.L rng=TC!A1:z200'

execute_unload "C:\graduation_project\ankarabursa_TT.gdx" TT.L
execute 'gdxxrw.exe C:\graduation_project\ankarabursa_TT.gdx o=C:\graduation_project\ankarabursa.xlsx var=TT.L rng=TT!A1:z200'

execute_unload "C:\graduation_project\ankarabursa_NS.gdx" NS.L
execute 'gdxxrw.exe C:\graduation_project\ankarabursa_NS.gdx o=C:\graduation_project\ankarabursa.xlsx var=NS.L rng=NS!A1:z200'

execute_unload "C:\graduation_project\ankarabursa_chargingtime.gdx" chargingtime.L
execute 'gdxxrw.exe C:\graduation_project\ankarabursa_chargingtime.gdx o=C:\graduation_project\ankarabursa.xlsx var=chargingtime.L rng=chargingtime!A1:z200'

execute_unload "C:\graduation_project\ankarabursa_SC.gdx" SC.L
execute 'gdxxrw.exe C:\graduation_project\ankarabursa_SC.gdx o=C:\graduation_project\ankarabursa.xlsx var=SC.L rng=SC!A1:z200'

execute_unload "C:\graduation_project\ankarabursa_TCT.gdx" TCT.L
execute 'gdxxrw.exe C:\graduation_project\ankarabursa_TCT.gdx o=C:\graduation_project\ankarabursa.xlsx var=TCT.L rng=totalchargingtime!A1:z200'

# The Python Code for the Interface Design:

from PyQt6 import QtCore, QtGui, QtWidgets
from PyQt6.QtWidgets import (QApplication, QWidget, QLabel, QFrame, 
                            QMessageBox, QVBoxLayout, QHBoxLayout, 
                            QGridLayout, QGroupBox, QPushButton, 
                            QTextEdit, QSpinBox, QComboBox, 
                            QProgressBar, QDialog, QScrollArea)
from PyQt6.QtGui import QFont, QPixmap
import matplotlib.pyplot as plt
from matplotlib.backends.backend_qt5agg import FigureCanvasQTAgg as FigureCanvas
import pandas as pd
import os
from datetime import datetime
import subprocess
import pandas as pd


class Ui_Form(object):
    def setupUi(self, Form):
        Form.setObjectName("Form")
        Form.resize(1000, 700)  # Slightly increased height for new input
        Form.setWindowTitle("EV Route Planner")
        
        # Main layout with some spacing
        self.verticalLayout = QVBoxLayout(Form)
        self.verticalLayout.setContentsMargins(0, 0, 0, 0)
        self.verticalLayout.setSpacing(0)
        
        # Create header banner with enhanced title
        self.header = QWidget(Form)
        self.header.setMinimumSize(QtCore.QSize(0, 100))
        self.header.setStyleSheet("background-color: #4CAF50;")
        
        # Main header layout
        header_layout = QVBoxLayout(self.header)
        header_layout.setContentsMargins(20, 10, 20, 10)
        header_layout.setSpacing(5)
        
        # Main title with larger, more prominent font
        self.title_label = QLabel("EV ROUTE PLANNER", self.header)
        self.title_label.setStyleSheet("""
            color: white;
            font-weight: bold;
            qproperty-alignment: AlignCenter;
        """)
        self.title_label.setFont(QFont("Segoe UI", 28, QFont.Weight.Bold))
        
        # Subtitle with improved styling
        self.subtitle = QLabel("Plan Your Electric Vehicle Journey Efficiently", self.header)
        self.subtitle.setStyleSheet("""
            color: #e8f5e9;
            font-size: 14pt;
            qproperty-alignment: AlignCenter;
            font-weight: 500;
        """)
        self.subtitle.setFont(QFont("Segoe UI", 12))
        
        # Add decorative line
        self.decorative_line = QFrame(self.header)
        self.decorative_line.setFrameShape(QFrame.Shape.HLine)
        self.decorative_line.setStyleSheet("""
            QFrame {
                color: rgba(255, 255, 255, 0.3);
                background-color: rgba(255, 255, 255, 0.3);
                border: none;
                height: 2px;
                margin: 5px 50px;
            }
        """)
        
        header_layout.addWidget(self.title_label)
        header_layout.addWidget(self.decorative_line)
        header_layout.addWidget(self.subtitle)
        
        self.verticalLayout.addWidget(self.header)

        # Main content widget with grey background
        self.contentWidget = QWidget(Form)
        self.contentWidget.setStyleSheet("background-color: #f5f5f5;")
        self.contentLayout = QVBoxLayout(self.contentWidget)
        self.contentLayout.setContentsMargins(20, 20, 20, 20)
        self.contentLayout.setSpacing(15)
        
        # Inputs group box - white with grey border
        self.groupBox = QGroupBox(parent=self.contentWidget)
        self.groupBox.setTitle("Journey Parameters")
        self.groupBox.setStyleSheet("""
            QGroupBox {
                border: 1px solid #e0e0e0;
                border-radius: 8px;
                margin-top: 10px;
                padding-top: 15px;
                font-weight: bold;
                color: #555;
                background-color: white;
                font-size: 12pt;
            }
            QGroupBox::title {
                subcontrol-origin: margin;
                left: 10px;
                padding: 0 5px;
                color: #555;
            }
        """)
        
        self.gridLayout = QGridLayout(self.groupBox)
        self.gridLayout.setVerticalSpacing(12)
        self.gridLayout.setHorizontalSpacing(20)

        # Battery level widget
        self.label_battery = QLabel("Battery Level (%)", parent=self.groupBox)
        self.battery_widget = QWidget(parent=self.groupBox)
        self.battery_layout = QHBoxLayout(self.battery_widget)
        self.battery_layout.setContentsMargins(0, 0, 0, 0)
        self.battery_layout.setSpacing(10)
        self.spin_battery = QSpinBox(parent=self.battery_widget)
        self.spin_battery.setRange(0, 100)
        self.spin_battery.setValue(50)
        self.spin_battery.setSuffix("%")
        self.progress_battery = QProgressBar(parent=self.battery_widget)
        self.progress_battery.setRange(0, 100)
        self.progress_battery.setValue(self.spin_battery.value())
        self.progress_battery.setTextVisible(True)
        self.progress_battery.setStyleSheet("""
            QProgressBar {
                border: 1px solid #e0e0e0;
                border-radius: 4px;
                text-align: center;
                height: 20px;
            }
            QProgressBar::chunk {
                background-color: #4CAF50;
                width: 10px;
            }
        """)
        self.battery_layout.addWidget(self.spin_battery)
        self.battery_layout.addWidget(self.progress_battery)
        self.gridLayout.addWidget(self.label_battery, 0, 0)
        self.gridLayout.addWidget(self.battery_widget, 0, 1)

        # Temperature input
        self.label_temp = QLabel("Temperature (°C)", parent=self.groupBox)
        self.spin_temp = QSpinBox(parent=self.groupBox)
        self.spin_temp.setRange(-20, 50)
        self.gridLayout.addWidget(self.label_temp, 1, 0)
        self.gridLayout.addWidget(self.spin_temp, 1, 1)

        # Desired battery level
        self.label_desired = QLabel("Desired Battery (%)", parent=self.groupBox)
        self.spin_desired = QSpinBox(parent=self.groupBox)
        self.spin_desired.setRange(0, 100)
        self.spin_desired.setValue(80)
        self.spin_desired.setSuffix("%")
        self.gridLayout.addWidget(self.label_desired, 2, 0)
        self.gridLayout.addWidget(self.spin_desired, 2, 1)

        # Battery capacity input (kWh)
        self.label_capacity = QLabel("Battery Capacity (kWh)", parent=self.groupBox)
        self.spin_capacity = QSpinBox(parent=self.groupBox)
        self.spin_capacity.setRange(20, 200)  # Typical EV battery range
        self.spin_capacity.setValue(60)       # Default to 60 kWh
        self.spin_capacity.setSuffix(" kWh")
        self.gridLayout.addWidget(self.label_capacity, 3, 0)
        self.gridLayout.addWidget(self.spin_capacity, 3, 1)

        # NEW: Optimization Objective selector
        self.label_alpha = QLabel("What's your preference", parent=self.groupBox)
        self.combo_alpha = QComboBox(parent=self.groupBox)
        self.combo_alpha.addItems([
            "Best Travel Time (α=0)", 
            "Minimum Cost (α=1)", 
            "Balanced (α=0.5)"
        ])
        self.combo_alpha.setCurrentIndex(2)  # Default to Balanced
        self.combo_alpha.setToolTip("Select the optimization priority for your journey")
        self.gridLayout.addWidget(self.label_alpha, 4, 0)
        self.gridLayout.addWidget(self.combo_alpha, 4, 1)

        # Start city
        self.label_start = QLabel("Start City", parent=self.groupBox)
        self.combo_start = QComboBox(parent=self.groupBox)
        self.combo_start.addItems(["Istanbul", "Ankara", "Antalya", "Kocaeli", "Konya", "Izmir", "Bursa"])
        self.gridLayout.addWidget(self.label_start, 5, 0)
        self.gridLayout.addWidget(self.combo_start, 5, 1)

        # Destination city
        self.label_end = QLabel("Destination City", parent=self.groupBox)
        self.combo_end = QComboBox(parent=self.groupBox)
        self.combo_end.addItems(["Istanbul", "Ankara", "Antalya", "Kocaeli", "Konya", "Izmir", "Bursa"])
        self.gridLayout.addWidget(self.label_end, 6, 0)
        self.gridLayout.addWidget(self.combo_end, 6, 1)
        
        # Start Planning button - Changed to light green
        self.btn_start = QPushButton("Start Planning", parent=self.groupBox)
        self.btn_start.setStyleSheet("""
            QPushButton {
                background-color: #4CAF50;
                color: white;
                border: none;
                border-radius: 4px;
                padding: 10px 15px;
                font-weight: bold;
                min-width: 160px;
                font-size: 11pt;
            }
            QPushButton:hover {
                background-color: #43A047;
            }
            QPushButton:pressed {
                background-color: #388E3C;
            }
        """)
        self.gridLayout.addWidget(self.btn_start, 7, 0, 1, 2)  # Span 2 columns

        self.contentLayout.addWidget(self.groupBox)

        # Results group box - Initially hidden
        self.groupBox_3 = QGroupBox(parent=self.contentWidget)
        self.groupBox_3.setTitle("Route Analysis Results")
        self.groupBox_3.setStyleSheet("""
            QGroupBox {
                border: 1px solid #e0e0e0;
                border-radius: 8px;
                margin-top: 10px;
                padding-top: 15px;
                font-weight: bold;
                color: #555;
                background-color: white;
                font-size: 12pt;
            }
        """)
        self.horizontalLayout = QHBoxLayout(self.groupBox_3)
        self.horizontalLayout.setSpacing(15)

        # Button column
        self.button_layout = QVBoxLayout()
        self.button_layout.setSpacing(10)

        # Style buttons with light green
        button_style = """
            QPushButton {
                background-color: #4CAF50;
                color: white;
                border: none;
                border-radius: 4px;
                padding: 10px 15px;
                font-weight: bold;
                min-width: 160px;
                font-size: 11pt;
            }
            QPushButton:hover {
                background-color: #43A047;
            }
            QPushButton:pressed {
                background-color: #388E3C;
            }
        """

        self.btn_stations = QPushButton("Number of Stations", parent=self.groupBox_3)
        self.btn_time = QPushButton("Total Travel Time", parent=self.groupBox_3)
        self.btn_charging = QPushButton("Charging Stations", parent=self.groupBox_3)
        self.btn_cost = QPushButton("Total Trip Cost", parent=self.groupBox_3)
        self.btn_all = QPushButton("Show All Results", parent=self.groupBox_3)
        self.btn_graph = QPushButton("View Route Graph", parent=self.groupBox_3)

        # Disable result buttons initially
        self.btn_stations.setEnabled(False)
        self.btn_time.setEnabled(False)
        self.btn_charging.setEnabled(False)
        self.btn_cost.setEnabled(False)
        self.btn_all.setEnabled(False)
        self.btn_graph.setEnabled(False)

        for btn in [self.btn_stations, self.btn_time, self.btn_charging, self.btn_cost, self.btn_all, self.btn_graph]:
            btn.setStyleSheet(button_style)
            self.button_layout.addWidget(btn)

        self.horizontalLayout.addLayout(self.button_layout)
        
        # Results display area
        self.label_all_output = QTextEdit(parent=self.groupBox_3)
        self.label_all_output.setPlaceholderText("Press 'Start Planning' to begin your route analysis")
        self.label_all_output.setReadOnly(True)
        self.label_all_output.setStyleSheet("""
            QTextEdit {
                background-color: white;
                border: 1px solid #e0e0e0;
                border-radius: 4px;
                padding: 12px;
                font-size: 11pt;
                color: #555;
            }
        """)
        self.horizontalLayout.addWidget(self.label_all_output)
        
        # Hide results section initially
        self.groupBox_3.setVisible(False)
        self.contentLayout.addWidget(self.groupBox_3)
        
        # Add status bar with light green
        self.status_bar = QFrame(Form)
        self.status_bar.setFrameShape(QFrame.Shape.StyledPanel)
        self.status_bar.setFrameShadow(QFrame.Shadow.Raised)
        self.status_bar.setStyleSheet("background-color: #4CAF50; color: white; padding: 5px;")
        self.status_label = QLabel("Ready to plan your EV journey!", self.status_bar)
        self.status_label.setStyleSheet("color: white;")
        
        status_layout = QHBoxLayout(self.status_bar)
        status_layout.addWidget(self.status_label)
        status_layout.setContentsMargins(10, 0, 10, 0)
        
        self.verticalLayout.addWidget(self.contentWidget)
        self.verticalLayout.addWidget(self.status_bar)
        
        # Connect signals
        self.spin_battery.valueChanged.connect(self.progress_battery.setValue)
        QtCore.QMetaObject.connectSlotsByName(Form)

class EVRoutePlanner(QWidget):
    def __init__(self):
        super().__init__()
        self.ui = Ui_Form()
        self.ui.setupUi(self)
        # Get the directory where the script is located
        self.script_dir = os.path.dirname(os.path.abspath(__file__))
        self.gams_path = self.script_dir  # Use script directory for GAMS files
        
        # Set the GAMS executable path
        self.gams_executable = r"C:\GAMS\win64\24.1\gamside.exe"
        
        # Connect buttons
        self.ui.btn_start.clicked.connect(self.start_planning)
        self.ui.btn_stations.clicked.connect(self.show_stations)
        self.ui.btn_time.clicked.connect(self.show_time)
        self.ui.btn_charging.clicked.connect(self.show_charging)
        self.ui.btn_cost.clicked.connect(self.show_cost)
        self.ui.btn_all.clicked.connect(self.show_all)
        self.ui.btn_graph.clicked.connect(self.show_graph)
    
    def get_excel_path(self):
        """Generate file path based on selected cities"""
        start = self.ui.combo_start.currentText()
        end = self.ui.combo_end.currentText()
        
        # Remove spaces and dashes, convert to lowercase
        start_clean = start.replace(" ", "").replace("-", "").lower()
        end_clean = end.replace(" ", "").replace("-", "").lower()
        
        filename = f"{start_clean}{end_clean}.xlsx"
        # Use script directory instead of old base_path
        return os.path.join(self.script_dir, filename)
    
    def get_station_names_path(self):
        """Generate station names file path based on selected cities"""
        start = self.ui.combo_start.currentText()
        end = self.ui.combo_end.currentText()
        
        # Remove spaces and dashes, convert to lowercase
        start_clean = start.replace(" ", "").replace("-", "").lower()
        end_clean = end.replace(" ", "").replace("-", "").lower()
        
        filename = f"{start_clean}{end_clean}_istasyonisimleri.xlsx"
        return os.path.join(self.script_dir, filename)
    
    def get_gams_filename(self):
        """Generate GAMS filename based on selected cities"""
        start = self.ui.combo_start.currentText()
        end = self.ui.combo_end.currentText()
        
        # Remove spaces and dashes, convert to lowercase
        start_clean = start.replace(" ", "").replace("-", "").lower()
        end_clean = end.replace(" ", "").replace("-", "").lower()
        
        return f"{start_clean}{end_clean}"
    
    def validate_file(self, path):
        """Check if file exists and is valid"""
        if not os.path.exists(path):
            QMessageBox.critical(
                self, 
                "File Not Found", 
                f"Route data file not found:\n{path}\n\n"
                "Please ensure you've selected valid cities and the route file exists."
            )
            self.ui.status_label.setText(f"Error: File not found - {os.path.basename(path)}")
            return False
        return True

    def run_gams_model(self):
        """Run the GAMS model for the selected route"""
        gams_base = self.get_gams_filename()
        
        # Try different file extensions for GAMS files
        possible_extensions = ['.gms', '.~gm']
        gams_filepath = None
        actual_filename = None
        
        for ext in possible_extensions:
            candidate = os.path.join(self.gams_path, gams_base + ext)
            if os.path.exists(candidate):
                gams_filepath = candidate
                actual_filename = os.path.basename(candidate)
                break
        
        # Validate that GAMS file exists
        if not gams_filepath:
            QMessageBox.critical(
                self, 
                "GAMS File Not Found", 
                f"GAMS model file not found for:\n{gams_base}\n\n"
                f"Tried extensions: {', '.join(possible_extensions)}\n"
                "Please ensure the model file exists for the selected route."
            )
            self.ui.status_label.setText(f"Error: GAMS file not found for {gams_base}")
            return False
        
        try:
            # Run the GAMS model
            self.ui.status_label.setText(f"Running GAMS model: {actual_filename}...")
            QApplication.processEvents()  # Update UI immediately
            
            # Use the full path to the GAMS executable
            subprocess.run([self.gams_executable, actual_filename], 
                           cwd=self.gams_path, 
                           check=True)
            
            self.ui.status_label.setText(f"GAMS model completed: {actual_filename}")
            return True
        except subprocess.CalledProcessError as e:
            QMessageBox.critical(
                self, 
                "GAMS Execution Error", 
                f"Failed to run GAMS model:\n{str(e)}\n\n"
                "Please check the GAMS installation and model file."
            )
            self.ui.status_label.setText(f"Error running GAMS: {actual_filename}")
            return False
        except Exception as e:
            QMessageBox.critical(
                self, 
                "GAMS Error", 
                f"Unexpected error running GAMS model:\n{str(e)}"
            )
            self.ui.status_label.setText(f"GAMS error: {actual_filename}")
            return False

    def read_excel_data(self):
        # Get dynamic path based on user selection
        self.excel_path = self.get_excel_path()
        
        # Validate file exists before processing
        if not self.validate_file(self.excel_path):
            return None
        
        data = {}
        try:
            # Read all sheets with proper index handling
            # For scalar values (single numbers)
            data['ns'] = self.read_scalar('NS')
            data['tt'] = self.read_scalar('TT')
            data['tc'] = self.read_scalar('TC')
            
            # For matrix data
            data['charge'] = self.read_matrix('charge')
            
            # Convert minutes to hours and minutes
            hours = data['tt'] // 60
            minutes = data['tt'] % 60
            data['tt_formatted'] = f"{int(hours)} hours {int(minutes)} minutes"
            
            return data
        except Exception as e:
            QMessageBox.critical(
                self, 
                "Data Read Error", 
                f"Error processing route data:\n{str(e)}\n\n"
                "Please ensure the Excel file format is correct."
            )
            self.ui.status_label.setText(f"Error reading data from {os.path.basename(self.excel_path)}")
            return None

    def read_scalar(self, sheet_name):
        """Read a single value from a sheet"""
        df = pd.read_excel(self.excel_path, sheet_name=sheet_name, header=None)
        
        # Find the first numeric value in the entire sheet
        for row in df.values:
            for cell in row:
                try:
                    return float(cell)
                except (ValueError, TypeError):
                    continue
        raise ValueError(f"No numeric value found in sheet {sheet_name}")

    def read_matrix(self, sheet_name):
        """Read matrix data from a sheet, handling index columns"""
        df = pd.read_excel(self.excel_path, sheet_name=sheet_name)
        
        # Remove any "Unnamed" index columns
        df = df.loc[:, ~df.columns.str.contains('^Unnamed')]
        
        # Convert to numeric, ignoring errors
        return df.apply(pd.to_numeric, errors='coerce')

    def get_station_names(self):
        """Load station names from the special file if available"""
        station_path = self.get_station_names_path()
        station_names = {}
        
        if not os.path.exists(station_path):
            # File doesn't exist - return empty dict
            return station_names
        
        try:
            # Read the Excel file
            df = pd.read_excel(station_path, sheet_name='Sayfa1', header=None)
            
            # Process each row starting from row index 2 (which is the 3rd row)
            for row_index in range(2, len(df)):
                row = df.iloc[row_index]
                
                # Skip empty rows
                if pd.isna(row[0]) or row[0] == '':
                    continue
                
                # Get segment number from column A
                segment = int(row[0])
                
                # Collect station names from columns B to K
                segment_stations = []
                for col_index in range(1, 11):  # Columns B to K (indexes 1 to 10)
                    if col_index < len(row) and not pd.isna(row[col_index]):
                        segment_stations.append(str(row[col_index]).strip())
                
                # Store in dictionary
                station_names[segment] = segment_stations
                
        except Exception as e:
            print(f"Error reading station names: {str(e)}")
        
        return station_names

    def start_planning(self):
        """Collect inputs, create include file, run GAMS"""
        # Get user inputs
        battery = self.ui.spin_battery.value()
        temp = self.ui.spin_temp.value()
        desired = self.ui.spin_desired.value()
        capacity = self.ui.spin_capacity.value()  # Get battery capacity
        
        # Get optimization objective (alpha)
        alpha_index = self.ui.combo_alpha.currentIndex()
        alpha_values = [0.0, 1.0, 0.5]  # Corresponding to combo box items
        alpha = alpha_values[alpha_index]

        # Prepare include file name based on selected cities
        start = self.ui.combo_start.currentText()
        end = self.ui.combo_end.currentText()
        start_clean = start.replace(" ", "").replace("-", "").lower()
        end_clean = end.replace(" ", "").replace("-", "").lower()
        base_name = "input_parameter"
        inc_filename = f"{base_name}.inc"
        inc_path = os.path.join(self.script_dir, inc_filename)

        try:
            # Create include file (var=value; format)
            with open(inc_path, 'w', encoding='utf-8') as f:
                f.write(f'Binit={battery};\n')
                f.write(f'OutsideT={temp};\n')
                f.write(f'Batterylevelf={desired};\n')
                f.write(f'Bmax={capacity};\n')  # Add battery capacity parameter
                f.write(f'Alpha={alpha};\n')    # NEW: Add alpha parameter

            # Show success and run GAMS after a short delay
            self.ui.status_label.setText(f"Created include file: {inc_filename}")
            QtCore.QTimer.singleShot(1500, self.run_gams_after_delay)

        except Exception as e:
            error_msg = f"Could not create include file:\n{str(e)}"
            QMessageBox.critical(self, "File Error", error_msg)
            self.ui.status_label.setText(f"Error: {error_msg}")

    def run_gams_after_delay(self):
        """Run GAMS after ensuring files are created"""
        try:
            if not self.run_gams_model():
                return
                
            # Hide the Start Planning button
            self.ui.btn_start.setVisible(False)
            
            # Show results section
            self.ui.groupBox_3.setVisible(True)
            
            # Enable result buttons
            self.ui.btn_stations.setEnabled(True)
            self.ui.btn_time.setEnabled(True)
            self.ui.btn_charging.setEnabled(True)
            self.ui.btn_cost.setEnabled(True)
            self.ui.btn_all.setEnabled(True)
            self.ui.btn_graph.setEnabled(True)
            
            # Clear previous results
            self.ui.label_all_output.clear()
            self.ui.label_all_output.setPlaceholderText("Select a result option to view details")
            
            # Show final status
            self.ui.status_label.setText(f"GAMS execution completed! Results available.")
            
            # Automatically show the route image
            self.show_route_image()
            
        except Exception as e:
            error_msg = f"GAMS execution failed:\n{str(e)}"
            QMessageBox.critical(self, "GAMS Error", error_msg)
            self.ui.status_label.setText(f"Error: {error_msg}")

    def show_route_image(self):
        """Display the route image after GAMS run."""
        start = self.ui.combo_start.currentText()
        end = self.ui.combo_end.currentText()
        start_clean = start.replace(" ", "").replace("-", "").lower()
        end_clean   = end.replace(" ", "").replace("-", "").lower()
        img_filename = f"{start_clean}{end_clean}.jpg"
        img_path = os.path.join(self.script_dir, img_filename)
        
        if not os.path.exists(img_path):
            QMessageBox.warning(
                self,
                "Image Not Found",
                f"Route image not found:\n{img_path}"
            )
            return

        # Create a dialog to display the image
        dlg = QDialog(self)
        dlg.setWindowTitle(f"Route Visualization: {start} → {end}")

        # Create layout and scroll area
        layout = QVBoxLayout(dlg)
        scroll = QScrollArea(dlg)
        scroll.setWidgetResizable(True)  # Allow scrolling for large images

        # Create label to display the image
        label = QLabel()
        pixmap = QPixmap(img_path)
        # make the label scale its pixmap to its own size
        label.setScaledContents(True)
        label.setPixmap(pixmap)
        label.setAlignment(QtCore.Qt.AlignmentFlag.AlignCenter)
        scroll.setWidget(label)

        layout.addWidget(scroll)
        
        # Show the dialog
        dlg.show()
        # ensure the dialog is sized before we force a redraw
        dlg.resize(800, 600)

    def show_stations(self):
        if data := self.read_excel_data():
            self.ui.label_all_output.setText(
                f"Number of Charging Stations: {int(data['ns'])}"
            )
            filename = os.path.basename(self.excel_path)
            self.ui.status_label.setText(f"Displayed stations from {filename}")

    def show_time(self):
        if data := self.read_excel_data():
            self.ui.label_all_output.setText(
                f"Total Travel Time: {data['tt_formatted']}"
            )
            filename = os.path.basename(self.excel_path)
            self.ui.status_label.setText(f"Displayed travel time from {filename}")

    def show_charging(self):
        if data := self.read_excel_data():
            # Read 'charge' sheet with first column as index (segments) and header row as stations
            charge_df = pd.read_excel(self.excel_path, sheet_name='charge', index_col=0)
            # Drop any unnamed columns
            charge_df = charge_df.loc[:, ~charge_df.columns.str.contains('^Unnamed')]
            charge_df = charge_df.apply(pd.to_numeric, errors='coerce')

            # Get station names if available
            station_names = self.get_station_names()

            stations = []
            for segment, row in charge_df.iterrows():
                for station, charge_value in row.items():
                    if charge_value > 0:
                        # Get station name if available
                        station_idx = int(station)
                        station_name = ""
                        
                        # Try to get station name from the names file
                        if station_names and segment in station_names:
                            segment_stations = station_names[segment]
                            if 0 < station_idx <= len(segment_stations):
                                station_name = segment_stations[station_idx-1] + " - "
                        
                        stations.append((int(segment), int(station), station_name, charge_value))

            stations.sort(key=lambda x: x[0])
            station_info = [
                f"Segment {seg}, Station {stat}: {name}{val:.2f} kWh"
                for seg, stat, name, val in stations
            ]

            self.ui.label_all_output.setText(
                "Charging Stations Along Route:\n\n" +
                ("\n".join(station_info) if station_info else "No charging stations used on this route")
            )
            filename = os.path.basename(self.excel_path)
            self.ui.status_label.setText(f"Displayed charging info from {filename}")

    def show_cost(self):
        if data := self.read_excel_data():
            self.ui.label_all_output.setText(
                f"Total Trip Cost: {data['tc']:.2f} TL"
            )
            filename = os.path.basename(self.excel_path)
            self.ui.status_label.setText(f"Displayed trip cost from {filename}")

    def show_all(self):
        if data := self.read_excel_data():
            current_battery = self.ui.spin_battery.value()
            current_temp = self.ui.spin_temp.value()
            desired_battery = self.ui.spin_desired.value()
            battery_capacity = self.ui.spin_capacity.value()
            start_city = self.ui.combo_start.currentText()
            end_city = self.ui.combo_end.currentText()
            
            # Get optimization objective description
            alpha_index = self.ui.combo_alpha.currentIndex()
            alpha_options = [
                "Best Travel Time (α=0)",
                "Minimum Cost (α=1)",
                "Balanced (α=0.5)"
            ]
            optimization_objective = alpha_options[alpha_index]

            try:
                # Read 'charge' sheet correctly
                charge_df = pd.read_excel(self.excel_path, sheet_name='charge', index_col=0)
                charge_df = charge_df.loc[:, ~charge_df.columns.str.contains('^Unnamed')]
                charge_df = charge_df.apply(pd.to_numeric, errors='coerce')

                # Get station names if available
                station_names = self.get_station_names()

                stations = []
                for segment, row in charge_df.iterrows():
                    for station, charge_value in row.items():
                        if charge_value > 0:
                            # Get station name if available
                            station_idx = int(station)
                            station_name = ""
                            
                            # Try to get station name from the names file
                            if station_names and segment in station_names:
                                segment_stations = station_names[segment]
                                if 0 < station_idx <= len(segment_stations):
                                    station_name = segment_stations[station_idx-1] + " - "
                            
                            stations.append((int(segment), int(station), station_name, charge_value))
                stations.sort(key=lambda x: x[0])
                station_info = [
                    f"  - Segment {seg}, Station {stat}: {name}{val:.2f} kWh"
                    for seg, stat, name, val in stations
                ]

                # Battery percentage sheet remains as before
                battery_info = "  Battery data not available"
                try:
                    battery_df = pd.read_excel(self.excel_path, sheet_name='batterypercentage')
                    battery_df = battery_df.loc[:, ~battery_df.columns.str.contains('^Unnamed')]
                    if not battery_df.empty:
                        battery_levels = battery_df.iloc[0].tolist()
                        battery_info = "\n".join(
                            [f"  - Segment {i+1}: {val:.1f}%"
                             for i, val in enumerate(battery_levels)]
                        )
                    else:
                        battery_info = "  No valid battery data found"
                except Exception as e:
                    battery_info = f"  Error reading battery data: {e}"

                output_text = (
                    "EV Route Planner - Complete Analysis\n\n"
                    f"Route: {start_city} → {end_city}\n"
                    f"Data Source: {os.path.basename(self.excel_path)}\n\n"
                    f"Current Battery: {current_battery}%\n"
                    f"Desired Battery: {desired_battery}%\n"
                    f"Temperature: {current_temp}°C\n"
                    f"Battery Capacity: {battery_capacity} kWh\n"
                    f"Optimization Objective: {optimization_objective}\n\n"
                    f"Number of Charging Stations: {int(data['ns'])}\n"
                    f"Total Travel Time: {data['tt_formatted']}\n"
                    f"Total Trip Cost: {data['tc']:.2f} TL\n\n"
                    "Charging Stations:\n" +
                    ("\n".join(station_info) if station_info else "  None") + "\n\n"
                    "Battery Levels by Segment:\n" +
                    battery_info
                )

                self.ui.label_all_output.setText(output_text)
                filename = os.path.basename(self.excel_path)
                self.ui.status_label.setText(f"Displayed full analysis from {filename}")

            except Exception as e:
                QMessageBox.critical(self, "Data Processing Error", f"Failed to process data:\n{e}")
                self.ui.status_label.setText("Error displaying results")

    def show_graph(self):
        # Get the current Excel file path
        excel_path = self.get_excel_path()
        if not os.path.exists(excel_path):
            QMessageBox.critical(self, "File Not Found", f"Excel file not found: {excel_path}")
            self.ui.status_label.setText("Error: Excel file not found for graph")
            return
        
        try:
            # Read battery percentage data
            battery_df = pd.read_excel(excel_path, sheet_name='batterypercentage')
            battery_values = battery_df.iloc[0].values  # Get the first row values
            
            # Read energy level data
            energy_df = pd.read_excel(excel_path, sheet_name='energylevel')
            energy_values = energy_df.iloc[0].values  # Get the first row values
            
            # Create distances based on the number of data points (each segment = 10km)
            num_points = len(battery_values)
            distances = [i * 10 for i in range(1, num_points + 1)]
            
            # Create the figure and axes
            fig, ax1 = plt.subplots(figsize=(8, 5))
            
            # Plot battery percentage
            ax1.plot(distances, battery_values, color='#4CAF50', marker='o', 
                    label='Battery Level (%)', linewidth=2)
            ax1.set_xlabel("Distance (km)", fontsize=10)
            ax1.set_ylabel("Battery Level (%)", color='#4CAF50', fontsize=10)
            ax1.tick_params(axis='y', labelcolor='#4CAF50')
            ax1.set_ylim(0, 100)  # Battery percentage range
            
            # Create a second y-axis for energy level
            ax2 = ax1.twinx()
            ax2.plot(distances, energy_values, color='#2196F3', marker='s', 
                    label='Energy Level (kWh)', linewidth=2)
            ax2.set_ylabel("Energy Level (kWh)", color='#2196F3', fontsize=10)
            ax2.tick_params(axis='y', labelcolor='#2196F3')
            
            # Set title
            start_city = self.ui.combo_start.currentText()
            end_city = self.ui.combo_end.currentText()
            plt.title(f"EV Route Analysis: {start_city} to {end_city}", fontsize=12, pad=20)
            
            # Add legend
            lines1, labels1 = ax1.get_legend_handles_labels()
            lines2, labels2 = ax2.get_legend_handles_labels()
            ax1.legend(lines1 + lines2, labels1 + labels2, loc='upper right')
            
            # Display the graph
            fig.tight_layout()
            plt.show()
            self.ui.status_label.setText("Displayed route analysis graph")
            
        except Exception as e:
            QMessageBox.critical(self, "Graph Error", f"Could not generate graph:\n{str(e)}")
            self.ui.status_label.setText("Error generating graph")
            
if __name__ == "__main__":
    import sys
    app = QApplication(sys.argv)
    
    # Set application style
    app.setStyle('Fusion')
    
    # Create and show main window
    window = EVRoutePlanner()
    window.show()
    sys.exit(app.exec())



