# projeto-filtros-passivos  
% =========================================================================  
% Projeto de Filtros Passivos para Crossover de Duas Vias  
% Autor: João Francisco Farias Cordeiro 
% Disciplina: Circuitos de Corrente Alternada - CC44CP  
% Professor: Lucas Bernardo Zilch  
% =========================================================================  
  
clear; clc; close all;  
  
% -------------------------------------------------------------------------  
% 1. Parâmetros de projeto    
% -------------------------------------------------------------------------   
R = 8;              % Impedância da carga (ohms)    
fc = 2000;          % Frequência de corte (Hz)  
wc = 2 * pi * fc;   % Frequência angular de corte (rad/s)  
  
% -------------------------------------------------------------------------  
% 2. Cálculo dos valores ideais (fórmulas do enunciado)  
% -------------------------------------------------------------------------  
% Filtro Passa-Baixas (LPF) – 2ª ordem Butterworth  
L_LPF_ideal = sqrt(2) * R / wc;          % Henry  
C_LPF_ideal = 1 / (sqrt(2) * wc * R);    % Farad  
  
% Filtro Passa-Altas (HPF) – 2ª ordem Butterworth  
L_HPF_ideal = R / (sqrt(2) * wc);        % Henry  
C_HPF_ideal = sqrt(2) / (wc * R);        % Farad  
  
fprintf('--- Valores Ideais ---\n');  
fprintf('LPF: L = %.3f mH, C = %.3f µF\n', L_LPF_ideal*1e3, C_LPF_ideal*1e6);  
fprintf('HPF: L = %.3f mH, C = %.3f µF\n', L_HPF_ideal*1e3, C_HPF_ideal*1e6);  
  
% -------------------------------------------------------------------------  
% 3. Tabelas de valores comerciais (conforme fornecido)  
% -------------------------------------------------------------------------  
% Indutores (mH) – Tabela 1   
indutores_mH = [0.10, 0.12, 0.15, 0.18, 0.22, 0.27, 0.33, 0.39, 0.47, 0.56, ...
                0.68, 0.82, 1.0, 1.2, 1.5, 1.8, 2.2, 2.7, 3.3, 3.9, 4.7, ...
                5.6, 6.8, 8.2, 10, 12, 15];  
indutores_H = indutores_mH * 1e-3;     % converter para Henry  
  
% Capacitores (µF) – Tabela 2  
capacitores_uF = [1.0, 1.2, 1.5, 1.8, 2.2, 2.7, 3.3, 3.9, 4.7, 5.6, 6.8, ...
                  8.2, 10, 12, 15, 18, 22, 27, 33, 39, 47, 56, 68, 82, 100];  
capacitores_F = capacitores_uF * 1e-6; % converter para Farad  
  
% -------------------------------------------------------------------------  
% 4. Função auxiliar para encontrar o valor mais próximo  
% -------------------------------------------------------------------------  
function valor_real = encontrar_mais_proximo(valor_ideal, vetor_comercial)  
    % Retorna o elemento do vetor_comercial com a menor diferença absoluta  
    [~, idx] = min(abs(vetor_comercial - valor_ideal));  
    valor_real = vetor_comercial(idx);  
end  
  
% -------------------------------------------------------------------------  
% 5. Seleção dos componentes reais  
% -------------------------------------------------------------------------  
L_LPF_real = encontrar_mais_proximo(L_LPF_ideal, indutores_H);  
C_LPF_real = encontrar_mais_proximo(C_LPF_ideal, capacitores_F);  
  
L_HPF_real = encontrar_mais_proximo(L_HPF_ideal, indutores_H);  
C_HPF_real = encontrar_mais_proximo(C_HPF_ideal, capacitores_F);  
  
fprintf('\n--- Valores Comerciais Selecionados ---\n');  
fprintf('LPF: L = %.3f mH, C = %.3f µF\n', L_LPF_real*1e3, C_LPF_real*1e6);  
fprintf('HPF: L = %.3f mH, C = %.3f µF\n', L_HPF_real*1e3, C_HPF_real*1e6);  
  
% -------------------------------------------------------------------------  
% 6. Cálculo das respostas em frequência  
% -------------------------------------------------------------------------  
freq = logspace(1, 5, 500);   % 10 Hz a 100 kHz (escala logarítmica)  
s = 1j * 2 * pi * freq;       % variável complexa s = jω  

% --- LPF Ideal (Butterworth) ---  
H_LPF_ideal = wc^2 ./ (s.^2 + sqrt(2)*wc*s + wc^2);  
mag_LPF_ideal_dB = 20*log10(abs(H_LPF_ideal));  
  
% --- LPF Real (topologia: L série, C paralelo com a carga R) ---  
% A função de transferência é: H(s) = 1 / (s^2*L*C + s*L/R + 1)  
H_LPF_real = 1 ./ (s.^2 * L_LPF_real * C_LPF_real + s * L_LPF_real/R + 1);  
mag_LPF_real_dB = 20*log10(abs(H_LPF_real));  

% --- HPF Ideal (Butterworth) ---  
H_HPF_ideal = s.^2 ./ (s.^2 + sqrt(2)*wc*s + wc^2);  
mag_HPF_ideal_dB = 20*log10(abs(H_HPF_ideal));  
  
% --- HPF Real (topologia: C série, L paralelo com a carga R) ---  
% A função de transferência é: H(s) = s^2*L*C / (s^2*L*C + s*L/R + 1)  
H_HPF_real = s.^2 * L_HPF_real * C_HPF_real ./ ...
             (s.^2 * L_HPF_real * C_HPF_real + s * L_HPF_real/R + 1);  
mag_HPF_real_dB = 20*log10(abs(H_HPF_real));  
  
% -------------------------------------------------------------------------  
% 7. Geração dos gráficos de Bode (magnitude)  
% -------------------------------------------------------------------------  
% Cria a pasta 'imagens' se não existir  
if ~exist('imagens', 'dir')  
    mkdir('imagens');  
end  
  
figure('Position', [100 100 900 700]);  
  
% Subplot LPF  
subplot(2,1,1);  
semilogx(freq, mag_LPF_ideal_dB, 'b-', 'LineWidth', 2); hold on;  
semilogx(freq, mag_LPF_real_dB, 'r--', 'LineWidth', 2);  
xlabel('Frequência (Hz)');  
ylabel('Magnitude (dB)');  
title('Filtro Passa-Baixas (LPF) – Comparação Ideal vs Real');  
legend('Ideal (Butterworth)', 'Real (componentes comerciais)', 'Location', 'southwest');  
grid on; xlim([10 100000]); ylim([-60 5]);  
line([fc fc], ylim, 'Color', 'k', 'LineStyle', ':', 'LineWidth', 1); % linha de corte  
  
% Subplot HPF  
subplot(2,1,2);  
semilogx(freq, mag_HPF_ideal_dB, 'b-', 'LineWidth', 2); hold on;  
semilogx(freq, mag_HPF_real_dB, 'r--', 'LineWidth', 2);  
xlabel('Frequência (Hz)');  
ylabel('Magnitude (dB)');  
title('Filtro Passa-Altas (HPF) – Comparação Ideal vs Real');  
legend('Ideal (Butterworth)', 'Real (componentes comerciais)', 'Location', 'southwest');  
grid on; xlim([10 100000]); ylim([-60 5]);  
line([fc fc], ylim, 'Color', 'k', 'LineStyle', ':', 'LineWidth', 1);  
  
% Salva a figura  
saveas(gcf, 'imagens/bode_ideal_vs_real.png');  
fprintf('\nGráfico salvo em "imagens/bode_ideal_vs_real.png"\n');  
  
% -------------------------------------------------------------------------  
% 8. Análise crítica – determinação da frequência de corte real  
% -------------------------------------------------------------------------  
% Conversão para magnitude linear (para encontrar -3 dB = 0.707)  
mag_LPF_real_lin = 10.^(mag_LPF_real_dB/20);  
mag_HPF_real_lin = 10.^(mag_HPF_real_dB/20);  
  
% LPF: frequência onde a magnitude cai abaixo de 1/sqrt(2)  
idx_lpf = find(mag_LPF_real_lin <= 1/sqrt(2), 1, 'first');  
if ~isempty(idx_lpf)  
    fc_LPF_real = freq(idx_lpf);  
else  
    fc_LPF_real = NaN;  
end  
  
% HPF: frequência onde a magnitude ultrapassa 1/sqrt(2) (subindo)  
idx_hpf = find(mag_HPF_real_lin >= 1/sqrt(2), 1, 'last');  
if ~isempty(idx_hpf)  
    fc_HPF_real = freq(idx_hpf);  
else  
    fc_HPF_real = NaN;  
end  
  
% -------------------------------------------------------------------------  
% 9. Exibição dos resultados da análise  
% -------------------------------------------------------------------------  
fprintf('\n--- Análise Crítica (Diferenças) ---\n');  
fprintf('Frequência de corte ideal (projetada): %.0f Hz\n', fc);  
fprintf('Frequência de corte real (LPF) : %.0f Hz (erro %.2f%%)\n', ...
        fc_LPF_real, (fc_LPF_real/fc - 1)*100);  
fprintf('Frequência de corte real (HPF) : %.0f Hz (erro %.2f%%)\n', ...
        fc_HPF_real, (fc_HPF_real/fc - 1)*100);  
fprintf('\nDesvios dos componentes:\n');  
fprintf('LPF: L = %.3f mH (ideal: %.3f) – variação %.1f%%\n', ...
        L_LPF_real*1e3, L_LPF_ideal*1e3, (L_LPF_real/L_LPF_ideal - 1)*100);  
fprintf('LPF: C = %.3f µF (ideal: %.3f) – variação %.1f%%\n', ...
        C_LPF_real*1e6, C_LPF_ideal*1e6, (C_LPF_real/C_LPF_ideal - 1)*100);  
fprintf('HPF: L = %.3f mH (ideal: %.3f) – variação %.1f%%\n', ...
        L_HPF_real*1e3, L_HPF_ideal*1e3, (L_HPF_real/L_HPF_ideal - 1)*100);  
fprintf('HPF: C = %.3f µF (ideal: %.3f) – variação %.1f%%\n', ...
        C_HPF_real*1e6, C_HPF_ideal*1e6, (C_HPF_real/C_HPF_ideal - 1)*100);  
![Gráfico de Bode](<img width="1407" height="1094" alt="bode_ideal_vs_real" src="https://github.com/user-attachments/assets/2c45b333-5bfc-4a76-8aa1-78c8f61ccc8b" />)  

fprintf('\n>>> Fim da execução.\n');  
