PWM
===

.. contents:: Contents
   :depth: 2
   :local:


1. Introduction
---------------

Pulse Width Modulation (PWM) is a technique used in embedded systems to generate analog-like signals using digital means. It works by varying the width of pulses (the "on" time) in a periodic waveform to encode information or control power delivered to a load.

.. image:: images/PWM-and-Duty-Cycle.jpg
   :width: 300
   :align: center
   :alt: PWM-and-Duty-Cycle

The amplitude of PWM is equal to VDD of microcontroller; i.e. for arduino controller, it is 5V; for stm32 controllers, it is 3.3V.

The average output of a PWM signal is given by :math:`V_{avg} = \frac{\text{Pulse Width}}{\text{Time Period}} \cdot V_{dd}`.


2. CubeMX Configuration
-----------------------

- Open CubeMX and `generate basic code <../basic_setup/generate_basic_code.html>`__ with:

  - microcontroller: ``stm32f407vgt6`` or board: ``STM32F407VG-DISC1``
  - project name: ``pwm_test``
  - Toolchain/IDE: ``Makefile``

- Go to  ``Pinout and Congiguration > Timers > TIM1``. Select ``PWM Generation CH1`` for **CHANNEL1**.

- Generate code.

.. image:: images/pwm.webp
   :width: 100%
   :align: center
   :alt: PWM Configuration


3. Code to Change Duty Cycle
----------------------------

- Navigate to ``Core > Src`` and open ``main.c``. 

- Add to the ``main()`` as:

   .. code-block:: c
   
      int main(void)
      {
      
        /* USER CODE BEGIN 1 */
      
        /* USER CODE END 1 */
      
        /* MCU Configuration--------------------------------------------------------*/
      
        /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
        HAL_Init();
      
        /* USER CODE BEGIN Init */
      
        /* USER CODE END Init */
      
        /* Configure the system clock */
        SystemClock_Config();
      
        /* USER CODE BEGIN SysInit */
      
        /* USER CODE END SysInit */
      
        /* Initialize all configured peripherals */
        MX_GPIO_Init();
        MX_TIM1_Init();
        /* USER CODE BEGIN 2 */
        HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_1);
        /* USER CODE END 2 */
      
        /* Infinite loop */
        /* USER CODE BEGIN WHILE */
        while (1)
        {
          float duty = 0.5f;
          htim1.Instance->CCR1 = (uint32_t)(duty * htim1.Instance->ARR);
          
          // You can also use the HAL function to set the duty cycle:
          // __HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_1, (uint32_t)(duty * htim1.Instance->ARR));
      
          /* USER CODE END WHILE */
      
          /* USER CODE BEGIN 3 */
        }
        /* USER CODE END 3 */


4. Test Output
--------------

- Connect the ``TIM1_CH1`` pin to positive of an ``LED`` and negative terminal to ``GND``.

- Change the value of ``duty`` to ``0``, ``0.1f``, ``0.8f`` and ``1.0f``, and observe the **brightness of LED**.

- Observe the output on **Oscilloscope**.

**Assignment**: Control the speed of a motor using any motor driver available.


5. Changing the Frequency of PWM
--------------------------------

The PWM frequency on an STM32 microcontroller can be calculated using the formula:

.. math::

   f_{\text{PWM}} = \frac{f_{\text{TIM}}}{\text{(ARR + 1) * (PSC + 1)}}

.. line-block::

   where,
   :math:`f_{\text{TIM}}`: Frequency of timer
   :math:`\text{ARR}`: Auto Reload Register Value
   :math:`\text{PSC}`: Prescaler


.. raw:: html

   <div style="margin-top: 20px; padding: 10px; border: 1px solid #ccc; border-radius: 5px; background-color: #f9f9f9;">
       <label for="freqTIM1" style="font-weight: bold;">f<sub>TIM</sub> (Hz):</label>
       <input type="number" id="freqTIM1" placeholder="Enter timer frequency (Hz)" style="margin-bottom: 10px; width: 100%; padding: 5px;">

       <label for="psc1" style="font-weight: bold;">PSC:</label>
       <input type="number" id="psc1" placeholder="Enter PSC value" style="margin-bottom: 10px; width: 100%; padding: 5px;">
       
       <label for="arr1" style="font-weight: bold;">ARR:</label>
       <input type="number" id="arr1" placeholder="Enter ARR value" style="margin-bottom: 10px; width: 100%; padding: 5px;">
       
       <button onclick="calculatePWM()" style="background-color: #4CAF50; color: white; border: none; padding: 10px; cursor: pointer; width: 100%;">Calculate PWM Frequency</button>
       
       <p id="result1" style="font-weight: bold; margin-top: 10px;"></p>
   </div>

   <script>
       function calculatePWM() {
           const freqTIM = parseFloat(document.getElementById('freqTIM1').value);
           const arr = parseFloat(document.getElementById('arr1').value);
           const psc = parseFloat(document.getElementById('psc1').value);
           
           if (isNaN(freqTIM) || isNaN(arr) || isNaN(psc) || freqTIM <= 0 || arr < 0 || psc < 0) {
               document.getElementById('result1').textContent = "Please enter valid positive numbers.";
               return;
           }
           
           const freqPWM = freqTIM / ((arr + 1) * (psc + 1));
           document.getElementById('result1').textContent = `Calculated PWM Frequency: ${freqPWM.toFixed(2)} Hz`;
       }
   </script>


To determine the **frequency of timer**, first you need to find out the **APB timer clock** in the **reference mannual** of the microcontroller. 

- On ``STM32CubeMX``, hover the cursor on ``Timers``. 

- Click the ``details and documentation`` and then ``Reference mannual``. Or click `reference mannual link <https://www.google.com/url?sa=t&source=web&rct=j&opi=89978449&url=https://www.st.com/resource/en/reference_manual/dm00031020-stm32f405-415-stm32f407-417-stm32f427-437-and-stm32f429-439-advanced-arm-based-32-bit-mcus-stmicroelectronics.pdf&ved=2ahUKEwiS2sXckL2IAxW3_aACHV3sIHsQFnoECBoQAQ&usg=AOvVaw2x8tbTRz8d9PfqXBk3qZ74>`__.

- You can find the **APB** number under **Memory and bus architecture**.

  .. image:: images/apb_check.png
     :width: 100%
     :align: center
     :alt: APB Peripheral Clock

Now on  ``STM32CubeMx``, go to ``Clock Configuration`` tab and check the target **APB** clock frequency.

 .. image:: images/clock_conf.png
    :width: 100%
    :align: center
    :alt: Clock Configuration

For this case, for ``TIM1``, the **APB2** timer clock is ``168MHz``.

Suppose, we want the PWM frequency to be ``50Hz``. For this we need to calculate and adjust the **ARR** and **PSC** values.

If :math:`f_{\text{TIM}}` is ``168MHz``, :math:`\text{PSC}` is ``167`` and :math:`f_{\text{PWM}}` is ``50Hz``, then :math:`\text{ARR}` will be:

.. math::

   \text{ARR} = \frac{f_{\text{TIM}}}{f_{\text{PWM}} \times (\text{PSC} + 1)} - 1

   = \frac{168 \times 10^6}{50 \times (167 + 1)} - 1

   = 19999


.. raw:: html

   <div style="margin-top: 20px; padding: 10px; border: 1px solid #ccc; border-radius: 5px; background-color: #f9f9f9;">
       <label for="freqPWM2" style="font-weight: bold;">f<sub>PWM</sub> (Hz):</label>
       <input type="number" id="freqPWM2" placeholder="Enter PWM frequency (Hz)" style="margin-bottom: 10px; width: 100%; padding: 5px;">

       <label for="freqTIM2" style="font-weight: bold;">f<sub>TIM</sub> (Hz):</label>
       <input type="number" id="freqTIM2" placeholder="Enter timer frequency (Hz)" style="margin-bottom: 10px; width: 100%; padding: 5px;">

       <label for="psc2" style="font-weight: bold;">PSC:</label>
       <input type="number" id="psc2" placeholder="Enter PSC value" style="margin-bottom: 10px; width: 100%; padding: 5px;">

       <button onclick="calculateARR()" style="background-color: #4CAF50; color: white; border: none; padding: 10px; cursor: pointer; width: 100%;">Calculate ARR</button>
       
       <p id="result2" style="font-weight: bold; margin-top: 10px;"></p>
   </div>

   <script>
       function calculateARR() {
           const freqPWM = parseFloat(document.getElementById('freqPWM2').value);
           const freqTIM = parseFloat(document.getElementById('freqTIM2').value);
           const psc = parseFloat(document.getElementById('psc2').value);

           if (isNaN(freqPWM) || isNaN(freqTIM) || isNaN(psc) || freqPWM <= 0 || freqTIM <= 0 || psc < 0) {
               document.getElementById('result2').textContent = "Please enter valid positive numbers.";
               return;
           }

           const ARR = (freqTIM / (freqPWM * (psc + 1))) - 1;

           if (ARR < 0) {
               document.getElementById('result2').textContent = "Calculated ARR is negative. Please check your inputs.";
           } else {
               document.getElementById('result2').textContent = `Calculated ARR: ${ARR.toFixed(2)}`;
           }
       }
   </script>


.. note::

   We chose :math:`\text{PSC}` as `167` because the :math:`f_{\text{TIM}}` is `168MHz`. So :math:`\frac{f_{\text{TIM}}}{\text{PSC} + 1}` will be `1MHz` for easy calculation.

.. note::

   To get good response from DC motors, higher PWM frequency is better but motordriver should be capable to handle that frequency. Lower frequency can make **tunning sound** from DC motors. I used ``8KHz`` pwm frequency for **planetary gear motors**. 

Go to ``Pinout & Configuration > Timers > TIM1 > Parameter Settings`` and set the **ARR** value to ``19999`` and **PSC** value to ``167``.

.. image:: images/freq_change.webp
   :width: 100%
   :align: center
   :alt: PWM Frequency Configuration

Generate the code, change the **duty cycle** between ``0%`` and ``100%``. Observe the output frequency on the oscilloscope.

**Assignment**: Control the angle of a servo motor.

.. Hint:: 

   For :math:`f_{\text{PWM}} = 50\,\text{Hz}`, time period :math:`T = 20\,\text{ms}`. 
   And :math:`1\,\text{ms} \equiv 0^\circ` and :math:`2\,\text{ms} \equiv 180^\circ`.

   .. code-block:: c

      duty = map<float>(angle, 0, 180, 1, 2);

      htim1.Instance->CCR1 = (uint32_t)(htim->Instance->ARR * duty / 20);
