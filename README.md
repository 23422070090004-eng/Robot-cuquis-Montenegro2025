# Robot-cuquis-Montenegro2025 DIARIO TECNICO
Para la estetica y el funcionamiento de este carro nos  inspiramos mas que nada en carros a control remoto comunes y corrientes empezando por la direccion para no tener muchas complicaciones tanto esteticas como tecnicas  en un futuro, esto fue algo en lo que todos los integrantes del equipo estuvimos de acuerdo para imprimir las piezas y armar la carcasa dandonos mas tiempo para la programacion de la camara y detalles que nos vallan surgiendo en el proceso, durante el proseso de estar modificando cosas como posicionamiento de camara usamos tornillos y tuercas, sinchos logrando que todo este estable para no tener detalles de movimiento durante las pruevas  
![carro vicion imageM (2)](https://github.com/user-attachments/assets/1b4526f2-0bf0-4b71-94ad-e2e2bf0d283f)
![carro vicion imageM (1)](https://github.com/user-attachments/assets/4e115b99-116b-4357-aa3d-36250d0a7169)
![carro vicion imagnM (1)](https://github.com/user-attachments/assets/9b477ea3-2cd1-4742-aba5-b3601197ea28)
![carro vicion imageM](https://github.com/user-attachments/assets/17f78e7b-8132-48f7-810f-0f4405db581d)
![carro vicion imagenM](https://github.com/user-attachments/assets/735ef91f-23f6-4ff3-8774-117a0b36f8aa)

Estas imagenes son prueba de la estetica del carro donde claramente se ve que la inspiracion para su creacion almenos en el aspecto estetico nos vasamos en coches de control remoto por su simplesa y estetica precentable.

![foto del equipo carro vicioM](https://github.com/user-attachments/assets/78504ece-18e0-43a4-860b-633b27372f1e)

![foto de compañero en incapacidad pero parte del equipo carro vicion M](https://github.com/user-attachments/assets/1fd9e799-6175-4194-94fd-d6ce89b73958)

en la imagen anterior estamos los integrantes de equipo 
Barcenas Reyes Betsabe 
Ortiz Ortiz Jose de Jesus
Rezendiz Moreno Jesus Emiliano
Servin Araujo Luis Fernando
Bailon Velazquez Sebastian Karol

codigos de programacion usados para el carro vicion 
# ================================
#       IMPORTACIÓN DE LIBRERÍAS
# ================================
import cv2                      # Manejo de cámara y visión artificial
import numpy as np             # Cálculos con matrices
from gpiozero import Servo, PWMLED, DigitalInputDevice   # Control de GPIO
from time import sleep          # Control de pausas

# ===========================================
#  AJUSTE FINO DEL CENTRO DEL SERVO (90° REAL)
# ===========================================
# Cambia este valor hasta que TU SERVO quede derecho
AJUSTE_CENTRO = -0.15           # <-- AJÚSTALO si aún no queda recto

# ===========================================
#          PIN DE ACTIVACIÓN (GPIO24)
# ===========================================
activacion = DigitalInputDevice(24)   # Si recibe 5V el robot funciona

# ===========================================
#               MOTOR PWM (GPIO23)
# ===========================================
motor = PWMLED(23)              # Motor controlado con PWM (0 a 1)

# ===========================================
#            SERVO DE DIRECCIÓN (GPIO14)
# ===========================================
servo = Servo(14, min_pulse_width=0.5/1000, max_pulse_width=2.5/1000)

# Poner el servo en posición recta (90° REAL)
servo.value = AJUSTE_CENTRO
sleep(1)    # Tiempo para estabilizar

# ===========================================
#              CONFIGURACIÓN CÁMARA
# ===========================================
cam = cv2.VideoCapture(0)       # Cámara USB /dev/video0
cam.set(3, 640)                 # Resolución horizontal
cam.set(4, 480)                 # Resolución vertical

# Verificar si la cámara funciona
ret, frame = cam.read()
if not ret:
    print("❌ No se pudo iniciar la cámara.")
    cam.release()
    exit()

print("Robot de visión iniciado...")

# ===========================================
#               BUCLE PRINCIPAL
# ===========================================
while True:
    ret, frame = cam.read()     # Captura frame de la cámara
    if not ret:
        break

    # ---------------------------------------------------------
    # 🚨 CONDICIÓN DE SEGURIDAD — ACTIVACIÓN POR GPIO24
    # ---------------------------------------------------------
    if not activacion.value:        # Si NO hay 5V en GPIO24
        servo.value = AJUSTE_CENTRO # Servo recto
        motor.value = 0             # Motor apagado

        # Aviso en pantalla
        cv2.putText(frame,
                    "⛔ ESPERANDO ACTIVACION EN GPIO24",
                    (10, 40),
                    cv2.FONT_HERSHEY_SIMPLEX,
                    0.7,
                    (0, 255, 255),
                    2)
        cv2.imshow("Robot Vision Future Engineers", frame)

        if cv2.waitKey(10) == 27:  # Tecla ESC
            break

        continue   # No ejecuta lo demás

    # =============================
    #  PROCESAMIENTO DE LA IMAGEN
    # =============================
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)     # A escala de grises
    blur = cv2.GaussianBlur(gray, (5, 5), 0)           # Suavizado
    _, thresh = cv2.threshold(blur, 80, 255, cv2.THRESH_BINARY_INV)

    superficie = (np.sum(thresh == 255) / thresh.size) * 100
    # Porcentaje de blanco = pared cercana

    # ===========================================
    #   LÓGICA DE MOVIMIENTO
    # ===========================================
    if superficie > 60:    # Pared a ~30 cm
        # ---- GIRO CONTROLADO A LA DERECHA ----
        servo.value = 1  # BETSABE (- DERECHA Y SIN EL - IZQUIERDA)
        motor.value = 0.22                 # El motor sigue avanzando

        sleep(4)   # <<<----- PAUSA para completar el giro
                      #        (ajusta entre 0.3 y 0.6)

        estado = "🚧 PARED CERCA: Girando derecha + Motor lento"
        color = (0, 0, 255)

    else:
        # ---- AVANCE RECTO ----
        servo.value = AJUSTE_CENTRO  # Servo recto real
        motor.value = 0.22            # Velocidad normal

        estado = "➡️ AVANZANDO RECTO"
        color = (0, 255, 0)

    # ===========================================
    #       MOSTRAR INFO EN PANTALLA
    # ===========================================
    cv2.putText(frame,
                estado,
                (10, 40),
                cv2.FONT_HERSHEY_SIMPLEX,
                0.7,
                color,
                2)

    cv2.imshow("Robot Vision Future Engineers", frame)

    # Salida con ESC
    if cv2.waitKey(10) == 27:
        break

# ===========================================
#           FINALIZACIÓN SEGURA
# ===========================================
cam.release()
motor.value = 0
servo.value = AJUSTE_CENTRO
cv2.destroyAllWindows()

url

La impresora que utilizamos para la imprecion de las piezas fue una impresora 3D de modelo Ende -3 V3 SE
esta fue la inica impresora que utilizamos sin cortadora laser ni mas maquinas extras.

![impresora 3D 2](https://github.com/user-attachments/assets/9bb228fb-da65-4a7a-baf8-50adcfb136e2)
![impresora 3D](https://github.com/user-attachments/assets/831f1ca2-4399-48fc-bea1-47d6231363bb)











