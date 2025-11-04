# Zadanie 1 – Mrugająca dioda i Watchdog (STM32F205RGT6)

---

## Konfiguracja zegara (Clock Configuration)
W zakładce Clock Configuration ustawiłem:
- źródło zegara na HSE (High Speed External) o częstotliwości 16 MHz  
- PLL tak, aby uzyskać SYSCLK = 96 MHz  
- dzielniki:
  - AHB Prescaler: /1 → HCLK = 96 MHz  
  - APB1 Prescaler: /4 → APB1 = 24 MHz (częstotliwość timera na tej magistrali to 48 MHz)  
  - APB2 Prescaler: /2 → APB2 = 48 MHz  

Dzięki temu timer TIM3 działa z częstotliwością 48 MHz.

---

## Konfiguracja GPIO
Ustawiłem pin PA5 jako GPIO Output.  
Do tego pinu podłączona jest dioda LED.  
Zmiana stanu pinu powoduje zapalenie lub zgaszenie diody.

---

## Konfiguracja timera TIM3
Jako źródło zegara timera ustawiłem Internal Clock.  
Timer skonfigurowałem tak, aby generował przerwanie co 0.1 sekundy.  
Użyłem wartości:
- Prescaler = 4798  
- Counter Period (Reload) = 999  

W ten sposób przerwanie występuje 10 razy na sekundę (10 Hz).  
W każdym przerwaniu dioda zmienia stan, więc pełny cykl trwa 0.2 sekundy, co daje częstotliwość mrugania 5 Hz.

---

## Konfiguracja Watchdoga (IWDG)
Włączyłem niezależny watchdog (Independent Watchdog).  
W pętli głównej programu regularnie odświeżam watchdoga funkcją HAL_IWDG_Refresh(&hiwdg), aby zapobiec jego zadziałaniu.

---

## Opis działania programu
1. Po uruchomieniu mikrokontrolera inicjalizowane są wszystkie peryferia: GPIO, Timer3 oraz Watchdog.  
2. W funkcji main wywołuję HAL_TIM_Base_Start_IT(&htim3), co uruchamia timer z przerwaniami.  
3. Za każdym razem, gdy licznik timera osiągnie wartość zadaną w Counter Period, wywoływana jest funkcja HAL_TIM_PeriodElapsedCallback().  
4. W tej funkcji zmieniam stan pinu PA5 za pomocą HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5).  
5. Dzięki temu dioda podłączona do PA5 miga z częstotliwością 5 Hz.  
6. W pętli while(1) program odświeża watchdoga, aby mikrokontroler nie został zresetowany.  

---

## Podsumowanie
Projekt realizuje zadanie mrugania diodą z częstotliwością 5 Hz z użyciem timera TIM3 oraz włącza i obsługuje watchdog IWDG.  
Konfiguracja została wykonana w STM32CubeIDE w oparciu o mikrokontroler STM32F205RGT6.
