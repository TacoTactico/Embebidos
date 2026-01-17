Links de Descarga y Documentacion: 
https://docs.espressif.com/projects/vscode-esp-idf-extension/en/latest/prerequisites.html
https://docs.espressif.com/projects/vscode-esp-idf-extension/en/latest/installation.html#installation


Módulo C para MicroPython en ESP32-C6 (Rainbow / NeoPixel)

Este proyecto muestra cómo crear e integrar un módulo en C para MicroPython en un ESP32-C6, usando ESP-IDF 5.5.
El ejemplo implementa un barrido de colores del NeoPixel.





Requisitos
Antes de comenzar, asegúrate de tener instalado:

    ✅ Visual Studio Code
    ✅ ESP-IDF 5.5 CMD
    ✅ Thonny





Paso 1: Crear el módulo

✅Abre la carpeta C:\Espressif\frameworks\esp-idf-v5.5.1\micropython\ports\esp32\modules en Visual Studio Code.

✅Dentro de modules, crea una carpeta con el nombre que desees (en este ejemplo: rainbow).

✅Dentro de la carpeta creada, crea los siguientes archivos, con las siguientes extensiones:

    ✅micropython.mk
    ✅rainbow.c






Paso 2: Código del módulo y archivo micropython.mk

✅Dentro del archivo de rainbow.c debes colocar el siguiente codigo de ejemplo:

                                #include "py/runtime.h"
                                #include "driver/gpio.h"
                                #include "led_strip.h"

                                static led_strip_handle_t strip;
                                static uint8_t hue = 0;

                                static void hsv_to_rgb(uint8_t h, uint8_t *r, uint8_t *g, uint8_t *b) {
                                    uint8_t region = h / 43;
                                    uint8_t remainder = (h - region * 43) * 6;

                                    uint8_t p = 0;
                                    uint8_t q = (255 * (255 - remainder)) >> 8;
                                    uint8_t t = (255 * remainder) >> 8;

                                    switch (region) {
                                        case 0: *r = 255; *g = t;   *b = p;   break;
                                        case 1: *r = q;   *g = 255; *b = p;   break;
                                        case 2: *r = p;   *g = 255; *b = t;   break;
                                        case 3: *r = p;   *g = q;   *b = 255; break;
                                        case 4: *r = t;   *g = p;   *b = 255; break;
                                        default:*r = 255; *g = p;   *b = q;   break;
                                    }
                                }

                                static mp_obj_t rainbow_init(mp_obj_t pin_obj) {
                                    int pin = mp_obj_get_int(pin_obj);

                                    led_strip_config_t strip_config = {
                                        .strip_gpio_num = pin,
                                        .max_leds = 1,
                                        .led_pixel_format = LED_PIXEL_FORMAT_GRB,
                                        .led_model = LED_MODEL_WS2812,
                                    };

                                    led_strip_rmt_config_t rmt_config = {
                                        .clk_src = RMT_CLK_SRC_DEFAULT,
                                        .resolution_hz = 10 * 1000 * 1000,
                                    };

                                    led_strip_new_rmt_device(&strip_config, &rmt_config, &strip);
                                    return mp_const_none;
                                }
                                static MP_DEFINE_CONST_FUN_OBJ_1(rainbow_init_obj, rainbow_init);

                                static mp_obj_t rainbow_step(void) {
                                    uint8_t r, g, b;
                                    hsv_to_rgb(hue++, &r, &g, &b);

                                    led_strip_set_pixel(strip, 0, r, g, b);
                                    led_strip_refresh(strip);
                                    return mp_const_none;
                                }
                                static MP_DEFINE_CONST_FUN_OBJ_0(rainbow_step_obj, rainbow_step);

                                static const mp_rom_map_elem_t rainbow_module_globals[] = {
                                    { MP_ROM_QSTR(MP_QSTR___name__), MP_ROM_QSTR(MP_QSTR_rainbow) },
                                    { MP_ROM_QSTR(MP_QSTR_init), MP_ROM_PTR(&rainbow_init_obj) },
                                    { MP_ROM_QSTR(MP_QSTR_step), MP_ROM_PTR(&rainbow_step_obj) },
                                };

                                static MP_DEFINE_CONST_DICT(rainbow_module_dict, rainbow_module_globals);

                                const mp_obj_module_t rainbow_user_cmodule = {
                                    .base = { &mp_type_module },
                                    .globals = (mp_obj_dict_t *)&rainbow_module_dict,
                                };

                                MP_REGISTER_MODULE(MP_QSTR_rainbow, rainbow_user_cmodule);


✅Dentro del archivo de micropython.mk debes colocar el siguiente codigo, el cual le indica a Micropython que compile el modulo:

                                RAINBOW_MOD_DIR := $(USERMOD_DIR)
                                SRC_USERMOD += $(RAINBOW_MOD_DIR)/rainbow.c
                                CFLAGS_USERMOD += -I$(RAINBOW_MOD_DIR)





Paso 3: Compilar Micropython

Abre ESP-IDF 5.5 CMD y ejecuta:
        
                                cd micropython\ports\esp32
                                rmdir /s /q build
                                idf.py set-target esp32c6
                                idf.py -DUSER_C_MODULES=main/modules build





Paso 4: Flashear el ESP32-C6

Conecta el ESP32 a tu computadora y desde ESP-IDF 5.5 CMD ejecuta:

                                idf.py flash





Paso 5: Probar el módulo en Thonny}

Abre Thonny y ejecuta:

                                import rainbow
                                rainbow.test()

Si todo sale bien se tendra que mostrar el siguiente mensaje:

                                Rainbow module OK