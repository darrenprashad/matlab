
%% Darren Prashad, MAE315, Intro Lab

% system variables
syms stress strain D F L0 T0 W0 

% D = Measured deformation of speciman
% F = Measured force of speciman 

stress = F/(W0*T0);
strain = D/L0;
youngsmodulus = (F*L0)/(W0*T0*D);

% uncertainties, the following were all given from the doc
u_L0 = 1/64;    % in
u_T0 = 0.001/2; % in (caliper)
u_W0 = 0.001/2; % in (caliper)
u_F = 0.012 * F;
u_D = 0.012 * D;

% STRESS & STRAIN zeroth order uncertainty
u_Stress = sqrt((diff(stress,F).*u_F).^2 + (diff(stress,W0).*u_W0).^2 + (diff(stress,T0).*u_T0).^2);
u_Strain = sqrt((diff(strain,D).*u_D).^2 + (diff(strain,L0).*u_L0).^2);

% ULTIMATE STRESS & STRAIN zeroth order uncertainty


% YOUNGS MODULUS zeroth order uncertainty
u_youngsmodulus = sqrt((diff(youngsmodulus,F)*u_F)^2 + (diff(youngsmodulus,L0)*u_L0)^2 + (diff(youngsmodulus,W0)*u_W0)^2 + (diff(youngsmodulus,T0)*u_T0)^2 +(diff(youngsmodulus,D)*u_D)^2 );

% import
DATA = readmatrix('/Users/darrenprashad/Documents/MATLAB/MAE315/Intro_specimen.txt');
D = DATA(:,1); % first index is row, second index is column
F = DATA(:,2);

W0 = 0.412; % in
T0 = 0.057; % in
L0 = 6.125; % in

% creates uncertainty values
U_STRESS = eval(u_Stress);
U_STRAIN = eval(u_Strain);

% creates stress strain values
STRESS = eval(stress);
STRAIN = eval(strain);
u_youngsMODULUS = eval(u_youngsmodulus);
u_youngsMODULUS = mean(u_youngsMODULUS)

% finds ULTIMATE POINT
ultimatePOINTY = max(STRESS)
ultimatePOINTindex = find(abs(STRESS-ultimatePOINTY) < 0.001);
ultimatePOINTX = STRAIN(ultimatePOINTindex, 1)

% ULTIMATE POINT ERROR VALUES
u_ultimatePOINTX = U_STRAIN(ultimatePOINTindex)
u_ultimatePOINTY = U_STRESS(ultimatePOINTindex)

% finds FRACTURE POINT
sz = size(STRESS);
length = sz(1,1);
fractureX = STRAIN(length)
fractureY = STRESS(length)

% finding yeild point / finding young modules / 0.2% offset
INDEX = 50;
p = polyfit(STRAIN(1:INDEX,1),STRESS(1:INDEX,1), 1);
youngsMODULUS = p(1,1)

OFFSETx = STRAIN(1:length,1);
OFFSETy = youngsMODULUS*(OFFSETx-0.002);

i = find(OFFSETy >= STRESS, 1);

OFFSETx = OFFSETx(1:i,1);
OFFSETy = OFFSETy(1:i,1);
YEILDx = OFFSETx(i,1)
YEILDy = OFFSETy(i,1)

step = 150;

hold on
% plot STRESS v. STRAIN
plot(STRAIN, STRESS, 'k', 'LineWidth',2)
ylim([0 ultimatePOINTY+10^3])

% plot YEILD POINT
plot(OFFSETx, OFFSETy, 'b', 'LineWidth',1)
plot(YEILDx, YEILDy, 'o', 'MarkerSize',10, 'MarkerFaceColor','b')

% plot ULTIMATE POINT
plot(ultimatePOINTX,ultimatePOINTY, 'o', 'MarkerSize',10, 'MarkerFaceColor','m' )

% plot FRACTURE POINT
plot(fractureX, fractureY, 'o', 'MarkerSize',10, 'MarkerFaceColor','r' )

% plot ERROR BARS
errorbar(STRAIN(1:step:end), STRESS(1:step:end), U_STRESS(1:step:end),'.', 'Color','m')
errorbar(STRAIN(1:step:end), STRESS(1:step:end), U_STRAIN(1:step:end),'.', 'horizontal', 'Color','b')

xlabel('Strain (in/in)')
ylabel('Stress (lbf/in^2)')
title('Stress v. Strain')
legend('Calculated Stress. v Strain', '0.2% Offset', 'Yeild Point', 'Ultimate Point', 'Fracture Point', 'Stress Err', 'Strain Err', 'Location', 'southeast', 'box', 'off' )


hold off
