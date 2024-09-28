from machine import Pin, time_pulse_us, ADC, Timer
import time

SOUND_SPEED = 340
TRIG_PULSE_DURATION_US = 10

DISTANCE_LIMIT = 20  # Corrected here
MQ_LIMIT = 20000

trig_pin = Pin(4, Pin.OUT)
echo_pin = Pin(5, Pin.IN)
gasSensor = ADC(Pin(28))
buzzer = Pin(22, Pin.OUT)
led = Pin(21, Pin.OUT)
hazardSwitch = Pin(20, Pin.IN)

led.value(1)

isHazard = False

def hazard(timer):
    led.toggle()
    buzzer.toggle()

hazardTimer = Timer()

while True:
    # Trigger the ultrasonic sensor
    trig_pin.value(0)
    time.sleep_us(5)
    trig_pin.value(1)
    time.sleep_us(TRIG_PULSE_DURATION_US)
    trig_pin.value(0)
   
    # Measure the duration of the echo
    ultrason_duration = time_pulse_us(echo_pin, 1, 30000)
    distance_cm = SOUND_SPEED * ultrason_duration / 20000
    print("mq135", gasSensor.read_u16())
    print(f"Distance : {distance_cm} cm")
    print(hazardSwitch.value())
   
    # Check if the distance or gas levels are hazardous
    if (0 < distance_cm < DISTANCE_LIMIT or gasSensor.read_u16() > MQ_LIMIT):
        led.toggle()
        buzzer.toggle()
        time.sleep(0.2)
   
    # Check the hazard switch state
    elif hazardSwitch.value() == 0:
        if isHazard:
            hazardTimer.deinit()
            isHazard = False
        else:
            hazardTimer.init(freq=2.5, mode=Timer.PERIODIC, callback=hazard)
            isHazard = True
   
    # Reset to normal state
    else:
        buzzer.value(0)
        led.value(1)
    time.sleep_ms(500)
