Полная инструкция: Создание прошивки для STM32 с выводом "kirill" в Renode
# 1. Установка необходимого ПО
Требования:
Visual Studio Code

Расширение PlatformIO IDE

Renode (скачать с официального сайта)

# 2. Создание проекта PlatformIO
В VS Code:
Откройте панель PlatformIO (иконка домика слева)

PIO Home → New Project

Настройки проекта:

Name: stm32_kirill

Board: BluePill F103C8

Framework: STM32Cube

Или через терминал:
bash
pio project init --board bluepill_f103c8 --project-dir stm32_kirill
cd stm32_kirill
# 3. Настройка platformio.ini
Замените содержимое файла platformio.ini:

ini
[env:bluepill_f103c8]
platform = ststm32
board = bluepill_f103c8
framework = stm32cube
build_flags = -D STM32F103xB
# 4. Создание прошивки
Создайте папку и файл:
`bash`
`mkdir src`
Содержимое src/main.c:
`c`
// Прошивка для вывода "kirill" через UART
`int main(void) {
    // Адреса регистров STM32F103
    volatile unsigned int* RCC_APB2ENR = (unsigned int*)0x40021018;
    volatile unsigned int* GPIOA_CRH = (unsigned int*)0x40010804;
    volatile unsigned int* USART1_BRR = (unsigned int*)0x40013808;
    volatile unsigned int* USART1_CR1 = (unsigned int*)0x4001380C;
    volatile unsigned int* USART1_SR = (unsigned int*)0x40013800;
    volatile unsigned int* USART1_DR = (unsigned int*)0x40013804;
    
 Включаем тактирование GPIOA и USART1
    *RCC_APB2ENR |= (1<<2) | (1<<14);
    
Настраиваем PA9 как TX (USART1)
    *GPIOA_CRH = (*GPIOA_CRH & ~0xFF0) | 0x490;
    
Настраиваем USART1: 115200 baud, включение
    *USART1_BRR = 0x0341;
    *USART1_CR1 = (1<<13) | (1<<3);
    
char message[] = "kirill\n";
    
  while(1) {
Отправляем сообщение
        for(int i = 0; message[i] != 0; i++) {
            Ждем готовности передатчика
            while(!(*USART1_SR & (1<<7)));
            Отправляем символ
            *USART1_DR = message[i];
        }
        Задержка ~1 секунда
        for(volatile int j = 0; j < 1000000; j++);
    }
}
# 5. Сборка проекта
В терминале PlatformIO:
bash
pio run -t clean
pio run
Проверка успешной сборки:
bash
dir .pio\build\bluepill_f103c8\firmware.elf
(Должен появиться файл firmware.elf)

# 6. Подготовка к эмуляции
Скопируйте ELF файл в простое место:
bash
copy .pio\build\bluepill_f103c8\firmware.elf C:\firmware.elf
# 7. Создание конфигурации Renode
Создайте файл kirill_config.resc:
python
using sysbus
mach create "stm32"

# Загрузка модели STM32F103
machine LoadPlatformDescription @platforms/cpus/stm32f103.repl

# Загрузка прошивки
sysbus LoadELF "C:/firmware.elf"

# Настройка UART вывода
showAnalyzer sysbus.usart1

# Запуск эмуляции
machine Start
# 8. Запуск в Renode
Способ 1: Через конфигурационный файл
bash
renode kirill_config.resc
Способ 2: Команды вручную
В окне Renode выполните:

python
mach create
machine LoadPlatformDescription @platforms/cpus/stm32f103.repl
sysbus LoadELF "C:/firmware.elf"
showAnalyzer sysbus.usart1
start
# 9. Ожидаемый результат
Откроется окно анализатора UART

Каждую секунду будет выводиться "kirill"

Эмуляция будет работать бесконечно

# 10. Возможные проблемы и решения
Проблема: "undefined reference to main"
Решение: Убедитесь, что в src/main.c есть функция main

Проблема: "multiple definition of SystemInit"
Решение: Используйте чистый код без заголовочных файлов

Проблема: Не выводится текст в Renode
Решение: Попробуйте sysbus.usart2 вместо sysbus.usart1

Проблема: Команды PlatformIO не найдены
Решение: Откройте терминал через PIO Core CLI в панели PlatformIO

# 11. Структура проекта (итоговая)
text
stm32_kirill/
├── src/
│   └── main.c
├── platformio.ini
├── kirill_config.resc
└── .pio/
    └── build/
        └── bluepill_f103c8/
            └── firmware.elf
Теперь ваша прошивка готова и будет выводить "kirill" в эмуляторе STM32!
