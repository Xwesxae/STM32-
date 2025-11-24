🚀 Полное руководство: Создание прошивки для STM32 с выводом "kirill" в эмуляторе Renode
📋 Содержание
Требования

Установка ПО

Создание проекта

Настройка конфигурации

Создание прошивки

Сборка проекта

Настройка эмуляции

Запуск в Renode

Устранение неисправностей

Структура проекта

🛠 Требования
Необходимое программное обеспечение:
Visual Studio Code - редактор кода

PlatformIO IDE - расширение для VS Code

Renode - фреймворк для эмуляции микроконтроллеров

Git (опционально) - для контроля версий

Поддерживаемое оборудование:
Целевая плата: STM32 BluePill (STM32F103C8T6)

Архитектура: ARM Cortex-M3

📥 Установка ПО
1. Установите Visual Studio Code
Скачайте с официального сайта

Установите стандартным способом

2. Установите расширение PlatformIO
Откройте VS Code

Перейдите в Extensions (Ctrl+Shift+X)

Найдите "PlatformIO IDE"

Нажмите Install

3. Установите Renode
Скачайте с официального сайта

Установите согласно инструкциям для вашей ОС

🎯 Создание проекта
Способ A: Через PlatformIO Home
Откройте панель PlatformIO (иконка 🏠 слева)

Нажмите PIO Home → New Project

Заполните параметры:

Name: stm32_kirill_emulator

Board: BluePill F103C8

Framework: STM32Cube

Location: выберите удобную папку

Способ B: Через терминал
bash
# Создание новой папки проекта
mkdir stm32_kirill_emulator
cd stm32_kirill_emulator

# Инициализация PlatformIO проекта
pio project init --board bluepill_f103c8 --framework stm32cube
⚙️ Настройка конфигурации
Файл platformio.ini
Замените содержимое файла platformio.ini в корне проекта:

ini
; PlatformIO Configuration for STM32 BluePill
[env:bluepill_f103c8]
platform = ststm32
board = bluepill_f103c8
framework = stm32cube

; Build configuration
build_flags = 
    -D STM32F103xB
    -D HAL_UART_MODULE_ENABLED

; Debug settings
debug_tool = stlink
upload_protocol = stlink

; Optimization
build_type = release
💻 Создание прошивки
1. Создайте структуру папок
bash
# В терминале PlatformIO выполните:
mkdir src
2. Создайте файл прошивки
Файл: src/main.c

c
/**
 * STM32 Firmware for Renode Emulator
 * Outputs "kirill" via UART every second
 * Board: STM32F103C8T6 (BluePill)
 */

// Direct register access - no external dependencies
int main(void) {
    // STM32F103 Register Definitions
    volatile unsigned int* RCC_APB2ENR  = (unsigned int*)0x40021018;  // Clock control
    volatile unsigned int* GPIOA_CRH    = (unsigned int*)0x40010804;  // Port A config
    volatile unsigned int* USART1_BRR   = (unsigned int*)0x40013808;  // Baud rate
    volatile unsigned int* USART1_CR1   = (unsigned int*)0x4001380C;  // Control
    volatile unsigned int* USART1_SR    = (unsigned int*)0x40013800;  // Status
    volatile unsigned int* USART1_DR    = (unsigned int*)0x40013804;  // Data
    
    // Enable clocks for GPIOA and USART1
    *RCC_APB2ENR |= (1 << 2) | (1 << 14);
    
    // Configure PA9 as TX pin (Alternate function output)
    *GPIOA_CRH = (*GPIOA_CRH & ~0x000000F0) | 0x00000090;
    
    // Configure USART1: 115200 baud, 8N1, Transmitter enabled
    *USART1_BRR = 0x0341;  // 115200 baud @ 8MHz
    *USART1_CR1 = (1 << 13) | (1 << 3);  // UE + TE
    
    // Message to transmit
    char message[] = "kirill\n";
    
    // Main program loop
    while(1) {
        // Transmit each character
        for(int i = 0; message[i] != '\0'; i++) {
            // Wait for transmit buffer empty
            while(!(*USART1_SR & (1 << 7)));
            // Send character
            *USART1_DR = message[i];
        }
        
        // Simple delay ~1 second
        for(volatile int delay = 0; delay < 1000000; delay++);
    }
    
    return 0;
}
🔨 Сборка проекта
1. Очистка предыдущих сборок
bash
pio run -t clean
2. Компиляция прошивки
bash
pio run
3. Проверка результатов
bash
# Проверьте создание ELF файла
dir .pio\build\bluepill_f103c8\firmware.elf

# Или для Linux/Mac:
ls -la .pio/build/bluepill_f103c8/firmware.elf
Ожидаемый вывод:

text
[SUCCESS] Took X.XX seconds
🎮 Настройка эмуляции
1. Подготовка файла прошивки
bash
# Скопируйте ELF файл в простое место
copy .pio\build\bluepill_f103c8\firmware.elf C:\firmware.elf

# Или для Linux/Mac:
cp .pio/build/bluepill_f103c8/firmware.elf /tmp/firmware.elf
2. Создание конфигурации Renode
Файл: renode_config.resc

python
# Renode Configuration for STM32 Kirill Emulator
# Description: Loads and runs STM32 firmware that outputs "kirill" via UART

using sysbus

# Create new machine instance
mach create "stm32_kirill"

# Load STM32F103 platform description
machine LoadPlatformDescription @platforms/cpus/stm32f103.repl

# Load compiled firmware
sysbus LoadELF "C:/firmware.elf"
# For Linux/Mac: sysbus LoadELF "/tmp/firmware.elf"

# Create and show UART analyzer for output
showAnalyzer sysbus.usart1

# Optional: Set up logging
logLevel -1

# Start emulation
echo "Starting STM32 emulation..."
echo "Expected output: 'kirill' every second"
machine Start
🚀 Запуск в Renode
Способ 1: Автоматический запуск
bash
renode renode_config.resc
Способ 2: Ручной ввод команд
Запустите Renode

Введите команды последовательно:

python
mach create
machine LoadPlatformDescription @platforms/cpus/stm32f103.repl
sysbus LoadELF "C:/firmware.elf"
showAnalyzer sysbus.usart1
start
Способ 3: Запуск с логами
bash
renode --console renode_config.resc
✅ Ожидаемый результат
После успешного запуска вы должны увидеть:

Окно Renode с логами запуска

Окно UART анализатора с текстовым выводом

Повторяющееся сообщение каждую секунду:

text
kirill
kirill
kirill
...
🔧 Устранение неисправностей
❌ Проблема: "undefined reference to main"
Решение: Убедитесь, что в src/main.c присутствует функция main

❌ Проблема: "multiple definition of SystemInit"
Решение: Используйте предоставленный код без заголовочных файлов

❌ Проблема: Команды PlatformIO не найдены
Решение:

Откройте терминал через PIO Core CLI

Или используйте: C:\Users\[user]\.platformio\penv\Scripts\platformio.exe

❌ Проблема: Нет вывода в Renode
Решение: Попробуйте другой UART:

python
showAnalyzer sysbus.usart2  # Вместо usart1
❌ Проблема: Ошибки пути в Renode
Решение: Используйте абсолютные пути или скопируйте файл в корень диска

❌ Проблема: Сборка завершается с ошибками
Решение:

bash
# Полная очистка и пересборка
pio run -t clean
pio run -v  # Подробный вывод
📁 Структура проекта
text
stm32_kirill_emulator/
│
├── src/
│   └── main.c                 # Исходный код прошивки
│
├── platformio.ini            # Конфигурация PlatformIO
├── renode_config.resc        # Конфигурация Renode
│
└── .pio/                     # Автогенерируемые файлы
    └── build/
        └── bluepill_f103c8/
            ├── firmware.elf  # Скомпилированная прошивка
            ├── firmware.bin
            └── firmware.hex
🎉 Поздравляем!
Вы успешно создали и запустили прошивку STM32 в эмуляторе Renode. Теперь вы можете:

✅ Модифицировать код для вывода другого текста

✅ Изменить частоту отправки сообщений

✅ Добавить дополнительную функциональность

✅ Использовать как основу для более сложных проектов
