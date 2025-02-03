Projeto de Temporizador de Um Disparo com Raspberry Pi Pico W
Descrição
Este projeto implementa um sistema de temporização para o acionamento de LEDs utilizando o microcontrolador Raspberry Pi Pico W. O sistema é ativado por um botão (pushbutton) e controla três LEDs (azul, vermelho e verde) com temporização de 3 segundos entre cada mudança de estado.

Componentes Utilizados
Microcontrolador Raspberry Pi Pico W
3 LEDs (azul, vermelho e verde)
3 Resistores de 330 Ω
Botão (Pushbutton)
Funcionamento
Quando o botão é pressionado, os três LEDs são ligados.
Após 3 segundos, o LED azul é desligado.
Após mais 3 segundos, o LED vermelho é desligado.
Após mais 3 segundos, o LED verde é desligado.
O botão só pode alterar o estado dos LEDs quando o último LED for desligado.
Implementação de debounce para evitar múltiplos acionamentos indesejados do botão.
Código
#include <stdio.h>
#include "pico/stdlib.h"
#include "hardware/timer.h"

#define LED_AZUL 11
#define LED_VERMELHO 12
#define LED_VERDE 13
#define BOTAO 5

#define INTERVALO_ALARME_MS 3000

bool temporizador_ativo = false;
int estado_led = 0;
absolute_time_t ultimo_tempo_botao;

bool turn_off_callback(struct repeating_timer *t) {
    if (estado_led == 0) {
        gpio_put(LED_AZUL, 0);
        estado_led++;
        add_alarm_in_ms(INTERVALO_ALARME_MS, turn_off_callback, NULL, false);
    } else if (estado_led == 1) {
        gpio_put(LED_VERMELHO, 0);
        estado_led++;
        add_alarm_in_ms(INTERVALO_ALARME_MS, turn_off_callback, NULL, false);
    } else {
        gpio_put(LED_VERDE, 0);
        temporizador_ativo = false;
    }
    return false;
}

void botao_callback(uint gpio, uint32_t events) {
    absolute_time_t agora = get_absolute_time();
    if (absolute_time_diff_us(ultimo_tempo_botao, agora) > 300000) { // 300 ms debounce
        if (!temporizador_ativo) {
            gpio_put(LED_AZUL, 1);
            gpio_put(LED_VERMELHO, 1);
            gpio_put(LED_VERDE, 1);
            estado_led = 0;
            temporizador_ativo = true;
            add_alarm_in_ms(INTERVALO_ALARME_MS, turn_off_callback, NULL, false);
        }
        ultimo_tempo_botao = agora;
    }
}

int main() {
    stdio_init_all();
    sleep_ms(2000);

    gpio_init(LED_AZUL);
    gpio_set_dir(LED_AZUL, GPIO_OUT);
    gpio_init(LED_VERMELHO);
    gpio_set_dir(LED_VERMELHO, GPIO_OUT);
    gpio_init(LED_VERDE);
    gpio_set_dir(LED_VERDE, GPIO_OUT);

    gpio_init(BOTAO);
    gpio_set_dir(BOTAO, GPIO_IN);
    gpio_pull_up(BOTAO);
    gpio_set_irq_enabled_with_callback(BOTAO, GPIO_IRQ_EDGE_FALL, true, &botao_callback);

    while (1) {
        sleep_ms(100);
    }
}
Experimento com BitDogLab
Utilize a Ferramenta Educacional BitDogLab para testar o código utilizando o LED RGB nos GPIOs 11, 12 e 13 e o Botão A no GPIO 5.