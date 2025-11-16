The **Bucher DCU** is a PMSM Vector Control Inverter. This repository contains a simulation of the inverter and a PDF describing the mathematical models used. Please see **Datasheet.pdf** for the Bucher DCU datasheet. 

 We implement the DCU as a \textit{Matlab Function Block}. All parameters are doubles. The load torque, torque command and speed command are double time series (2xN matrix) and hv\_ok is a boolean double. \textbf{Units are: seconds, amps, volts, ohms, henries, farads, rad/s, Nm and kg\,m$^2$}. Please copy the DCU Implementation (excluding the demonstration runner at the top) within the Bucher\_DCU\_MLX\_Demo.mlx file into a Matlab Function block in your Simulink Simulation. You can use the demonstration runner within Bucher\_DCU\_MLX\_Demo.mlx to play around with the DCU and potentially debug it if you choose to modify the script.

\begin{table}[h!]
\centering
\small
\begin{tabular}{|l|l|}
\hline
\textbf{Parameter} & \textbf{Definition} \\
\hline
\multicolumn{2}{|l|}{\textbf{Simulation}} \\
\hline
Ts              & sample time \\
Tfinal          & sim duration \\
\hline
\multicolumn{2}{|l|}{\textbf{PMSM Electrical}} \\
\hline
p               & pole pairs \\
Rs              & stator resistance \\
Ld              & d-axis inductance \\
Lq              & q-axis inductance \\
psi\_f          & PM flux linkage \\
\hline
\multicolumn{2}{|l|}{\textbf{Current, Torque, Speed Limits}} \\
\hline
Imax            & current limit \\
diq\_slew       & $i_q$ slew limit \\
did\_slew       & $i_d$ slew limit \\
Tmax\_reg       & regen torque limit \\
T\_rated        & rated torque \\
T\_fac          & torque factor \\
w\_max          & speed limit \\
acc\_max        & accel limit \\
dec\_max        & decel limit \\
\hline
\multicolumn{2}{|l|}{\textbf{DC Link}} \\
\hline
Vdc\_nom        & nominal DC-link \\
Vdc\_max        & max DC-link \\
Vdc\_min        & min DC-link \\
Cdc             & DC-link capacitance \\
Rsrc            & source resistance \\
dv\_max         & voltage slew limit \\
\hline
\multicolumn{2}{|l|}{\textbf{Current, Speed PI Gains}} \\
\hline
Kp\_d           & $i_d$ PI $K_p$ \\
Ki\_d           & $i_d$ PI $K_i$ \\
Kp\_q           & $i_q$ PI $K_p$ \\
Ki\_q           & $i_q$ PI $K_i$ \\
decouple\_k     & $dq$ decoupling gain \\
Kp\_w           & speed PI $K_p$ \\
Ki\_w           & speed PI $K_i$ \\
\hline
\multicolumn{2}{|l|}{\textbf{Field Weakening}} \\
\hline
FW\_Kp          & FW PI $K_p$ \\
FW\_Ti          & FW PI $T_i$ \\
vfac            & voltage fraction \\
id\_fac         & $|i_d|/I_\text{max}$ limit \\
FW\_on          & FW on threshold \\
FW\_off         & FW off threshold \\
\hline
\multicolumn{2}{|l|}{\textbf{Regen Controller}} \\
\hline
omega\_regen\_min & regen stops below this speed \\
Vp\_vdc         & regen PI $K_p$ \\
Tn\_vdc         & regen PI $T_i$ \\
\hline
\multicolumn{2}{|l|}{\textbf{Mechanical}} \\
\hline
J               & rotor inertia \\
B               & viscous friction \\
T\_coulomb      & Coulomb friction \\
\hline
\multicolumn{2}{|l|}{\textbf{Resolver / Inner Mode}} \\
\hline
mode\_inner     & inner angle mode \\
pos\_offset     & angle offset \\
pole\_pairs\_ratio & pole pair ratio \\
alpha\_res      & resolver low pass filter gain \\
hv\_ok          & HV enable (safety switch) \\
\hline
\end{tabular}
\caption{Simulink Parameters}
\label{tab:param_minimal}
\end{table}

**Background**
An inverter applies a 3-phase AC voltage to three windings in the stator. This creates a magnetic pole pair every 120 degrees, creating a rotating magnetic field vector with speed $\omega_e$; the poles of the rotor (rotating shaft) follow this vector with $\omega_m$. We can transform the 3-phase flux $abc$ into 2 dimensions $dq$: a $d$-axis pointing down a rotor north pole and a $q$-axis perpendicular to $d$, which creates torque. Since each stator current produces a magnetic field in one of the three directions $abc$, we project the current vector on $d$, $q$ (based on each current's influence on flux in these directions). The inductances $L_d$ and $L_q$ represent how easy it is to add flux in either direction. Since $i_d$ has no effect on torque, we usually just set it to 0 (unless $dq$ inductances are not equal, in which case it has influence). If DC voltage is too high, we decrease $i_d$ to reduce back EMF in $q$ and thus require a lower voltage to yield the same torque. This is called Field Weakening. To control our motor's speed or torque, we demand a certain torque value; either directly through torque mode or indirectly by modifying demanded torque using a PI loop until we get a desired speed.

<img width="606" height="430" alt="image" src="https://github.com/user-attachments/assets/e462f799-7751-46b4-88ec-67e3a61488b6" />
