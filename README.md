Generador de Pasillo Animado en Blender
Descripción General
Este script de Python para Blender genera automáticamente un pasillo curvo con una cámara animada que recorre toda la estructura. Es perfecto para crear escenas cinemáticas, visualizaciones arquitectónicas o fondos para animaciones.
El pasillo consta de bloques alternados en colores oscuro y naranja a ambos lados de un suelo blanco, siguiendo una trayectoria sinusoidal suave. La cámara sigue automáticamente este camino a lo largo de 300 frames.

Características Principales

Generación automática de pasillo con curvas suaves
Cámara animada que sigue el recorrido completo
Colores alternados en los bloques (patrón ajedrezado)
Iluminación profesional con luz solar y luces puntuales
Totalmente parametrizable (ancho, longitud, curvas, velocidad)
Listo para renderizar directamente en Blender


Cómo Usar
Paso 1: Preparar Blender

Abre Blender (versión 2.8 o superior recomendada)
Ve a la pestaña Scripting en la parte superior
Crea un nuevo archivo de texto o abre uno existente

Paso 2: Ejecutar el Script

Copia y pega el código completo en el editor de texto
Presiona Alt + P o haz clic en Run Script
El pasillo se generará automáticamente

Paso 3: Ver la Animación

Presiona NUMPAD 0 para ver desde la cámara
Presiona Z y selecciona Material Preview o Rendered
Presiona ESPACIO para reproducir la animación
La cámara recorrerá todo el pasillo automáticamente


Parámetros Configurables
Puedes personalizar el pasillo modificando estas variables en la sección PASO 3:
pythonancho_pasillo = 3.5          # Ancho entre bloques (más grande = pasillo más amplio)
num_bloques = 40             # Cantidad de bloques a lo largo del camino
longitud_total = 50          # Distancia total del recorrido
amplitud_curva = 8           # Qué tan pronunciadas son las curvas (0 = recto)
frecuencia_curva = 2         # Número de ondulaciones en el recorrido
Para ajustar la velocidad de la cámara, modifica en PASO 8:
pythonnum_frames = 300  # Más frames = cámara más lenta

Explicación Detallada del Código
PASO 1: Función de Creación de Materiales
pythondef crear_material(nombre, color_rgb):
    mat = bpy.data.materials.new(name=nombre)
    mat.diffuse_color = (*color_rgb, 1.0)
    return mat
¿Qué hace?
Esta función crea materiales (colores) que se aplicarán a los objetos 3D. Recibe un nombre y un color en formato RGB (valores de 0 a 1 para rojo, verde y azul).
Ejemplo:

(1.0, 0.0, 0.0) = rojo puro
(0.1, 0.1, 0.1) = gris oscuro
(1.0, 1.0, 1.0) = blanco


PASO 2: Limpieza del Entorno
pythonbpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete()
¿Qué hace?
Elimina todos los objetos existentes en la escena para empezar con un lienzo limpio. Esto evita que objetos anteriores interfieran con el nuevo pasillo.

PASO 3: Definición de Materiales
pythonmat_pared_oscura = crear_material("ParedOscura", (0.1, 0.1, 0.1))
mat_pared_naranja = crear_material("ParedNaranja", (0.8, 0.3, 0.1))
mat_suelo_blanco = crear_material("SueloBlanco", (1.0, 1.0, 1.0))
¿Qué hace?
Crea tres materiales que se usarán en el pasillo:

Pared Oscura: Gris casi negro para contraste
Pared Naranja: Color naranja cálido para dinamismo
Suelo Blanco: Base limpia que refleja luz


PASO 4: Configuración de Parámetros
pythonancho_pasillo = 3.5
num_bloques = 40
longitud_total = 50
amplitud_curva = 8
frecuencia_curva = 2
¿Qué hace?
Define las dimensiones y características del pasillo:

ancho_pasillo: Espacio entre las dos filas de bloques
num_bloques: Cuántos bloques habrá en cada lado
longitud_total: Distancia desde el inicio hasta el final
amplitud_curva: Qué tan lejos se desvía lateralmente (en unidades)
frecuencia_curva: Cuántas "olas" tendrá el recorrido


PASO 5: Generar Puntos del Recorrido
pythonfor i in range(num_bloques):
    t = i / (num_bloques - 1)
    y = t * longitud_total
    x = amplitud_curva * math.sin(frecuencia_curva * math.pi * t)
    
    dx_dt = amplitud_curva * frecuencia_curva * math.pi * math.cos(frecuencia_curva * math.pi * t)
    dy_dt = longitud_total
    angulo_rotacion = math.atan2(dx_dt, dy_dt)
¿Qué hace?
Calcula las posiciones (x, y) de cada bloque usando una función sinusoidal para crear curvas suaves:

t: Progreso del 0 al 1 (0% al 100% del recorrido)
y: Avanza linealmente hacia adelante
x: Se mueve lateralmente siguiendo un patrón de onda (seno)
angulo_rotacion: Calcula cómo deben rotar los bloques para seguir la curva tangencialmente

Analogía: Imagina dibujar una serpiente en un papel. La "y" es qué tan abajo está cada segmento, y la "x" es qué tanto se curva a los lados.

PASO 6: Crear Bloques del Pasillo
pythonfor i, pos in enumerate(posiciones_camino):
    # Calcular posición perpendicular
    dx = math.cos(angulo)
    dy = -math.sin(angulo)
    
    # BLOQUE IZQUIERDO
    x_izq = x_centro - dx * (ancho_pasillo / 2)
    y_izq = y_centro - dy * (ancho_pasillo / 2)
    bpy.ops.mesh.primitive_cube_add(location=(x_izq, y_izq, 0.5))
¿Qué hace?
Para cada punto del camino:

Calcula el vector perpendicular: Determina hacia dónde están los "lados" del pasillo
Crea bloque izquierdo y derecho: Coloca cubos a ambos lados del centro
Rota los bloques: Los orienta para que sigan la dirección de la curva
Alterna colores: Usa un patrón de ajedrez (bloque 1 oscuro, bloque 2 naranja, etc.)
Crea el suelo: Coloca un plano blanco entre los bloques

Resultado: Un "túnel" con paredes coloridas que sigue la curva calculada.

PASO 7: Crear Curva Bezier para la Cámara
pythonbpy.ops.curve.primitive_bezier_curve_add(location=(0, 0, 0))
path_curve = bpy.context.active_object

spline = curve_data.splines.new('BEZIER')
spline.bezier_points.add(num_puntos_camara - 1)

for i in range(num_puntos_camara):
    point = spline.bezier_points[i]
    point.co = (x, y, altura_camara)
    point.handle_left_type = 'AUTO'
    point.handle_right_type = 'AUTO'
¿Qué hace?
Crea una curva Bezier invisible que actúa como "riel" para la cámara:

num_puntos_camara: Usa el doble de bloques para movimiento más suave
Puntos Bezier: Más sofisticados que líneas rectas, crean transiciones suaves
handle_type = 'AUTO': Blender calcula automáticamente las curvas más suaves
altura_camara = 2.0: La cámara flota 2 unidades sobre el suelo

Analogía: Es como poner rieles invisibles para un carrito de montaña rusa.

PASO 8: Configurar Cámara con Follow Path
pythonfollow_path = camera.constraints.new(type='FOLLOW_PATH')
follow_path.target = path_curve
follow_path.use_curve_follow = True
follow_path.forward_axis = 'FORWARD_Y'
follow_path.up_axis = 'UP_Z'
¿Qué hace?
Agrega un "constraint" (restricción) a la cámara que la obliga a seguir la curva:

target = path_curve: Le dice a la cámara qué curva seguir
use_curve_follow = True: La cámara apunta en la dirección del movimiento
forward_axis = 'FORWARD_Y': El frente de la cámara es el eje Y
up_axis = 'UP_Z': El techo de la escena es el eje Z

Resultado: La cámara "mira hacia adelante" mientras se mueve, como si fueras tú caminando por el pasillo.

PASO 9: Animar el Movimiento
pythonnum_frames = 300
follow_path.offset = -0      # Inicio (frame 1)
follow_path.keyframe_insert(data_path="offset", frame=1)

follow_path.offset = -100    # Final (frame 300)
follow_path.keyframe_insert(data_path="offset", frame=num_frames)
¿Qué hace?
Crea la animación usando keyframes (puntos clave):

Frame 1: La cámara está al 0% de la curva (inicio)
Frame 300: La cámara está al 100% de la curva (final)
interpolation = 'LINEAR': Velocidad constante (sin aceleraciones)

Dato: 300 frames ÷ 24 FPS = 12.5 segundos de animación.

PASO 10: Iluminación
python# Luz solar general
bpy.ops.object.light_add(type='SUN', location=(0, 25, 30))
sol.data.energy = 1.5

# Luces puntuales cada 5 bloques
for i in range(0, len(posiciones_camino), 5):
    bpy.ops.object.light_add(type='POINT', location=(pos['x'], pos['y'], 4))
    luz.data.energy = 500
¿Qué hace?
Añade dos tipos de iluminación:

Luz Solar (SUN): Iluminación general desde arriba, simula luz ambiente
Luces Puntuales (POINT): Colocadas cada 5 bloques, crean atmósfera y profundidad

Efecto: El pasillo tiene iluminación profesional sin necesidad de configuración manual.

PASO 11: Ocultar la Curva
pythonpath_curve.hide_render = True
path_curve.hide_viewport = True
¿Qué hace?
Hace invisible la curva guía para que:

No aparezca en el render final
No moleste en la vista del viewport

La curva sigue funcionando, pero nadie la ve.

Conceptos Clave Explicados
¿Qué es una Curva Bezier?
Es un tipo de curva matemática que crea transiciones suaves entre puntos. Blender usa "handles" (manijas) para controlar la curvatura automáticamente.
¿Qué es un Constraint?
Una restricción que le dice a un objeto cómo debe comportarse. En este caso, "Follow Path" hace que la cámara siga la curva sin tener que animar manualmente cada frame.
¿Por qué usar math.sin() y math.cos()?
Estas funciones trigonométricas crean patrones de onda perfectos. El seno genera el movimiento lateral, y el coseno calcula la dirección de la tangente.
¿Qué es atan2()?
Calcula el ángulo de rotación necesario para que un objeto apunte en cierta dirección. Es como usar una brújula matemática.

Personalizaciones Avanzadas
Cambiar los colores:
pythonmat_pared_naranja = crear_material("ParedNaranja", (0.0, 0.5, 1.0))  # Azul
Hacer el pasillo recto:
pythonamplitud_curva = 0  # Sin curvas laterales
Recorrido más rápido:
pythonnum_frames = 150  # La mitad del tiempo (6.25 segundos)
Más ondulaciones:
pythonfrecuencia_curva = 4  # Doble de curvas

Resolución de Problemas
"La cámara no se mueve"

Asegúrate de presionar ESPACIO para reproducir la animación
Verifica que estás en vista de cámara (NUMPAD 0)

"Los bloques se ven raros"

Prueba reducir amplitud_curva si las curvas son muy pronunciadas
Aumenta num_bloques para transiciones más suaves

"No veo los colores"

Cambia a vista Material Preview (Z → Material Preview)
O renderiza la escena (F12)


Requisitos

Blender: Versión 2.8 o superior
Python: Incluido con Blender (no necesitas instalarlo)
Sistema: Windows, macOS o Linux


Licencia
Este proyecto es de código abierto. Siéntete libre de usarlo, modificarlo y compartirlo.

Contribuciones
¿Tienes ideas para mejorar el script? Las contribuciones son bienvenidas!

Haz un fork del repositorio
Crea una rama para tu mejora
Envía un pull request


Recursos Adicionales

Documentación oficial de Blender Python API
Tutorial de Blender Scripting
Comunidad de Blender en Stack Exchange


Código Completo
pythonimport bpy
import math

def crear_material(nombre, color_rgb):
    """
    Crea un material básico con un color específico usando el modelo RGB
    """
    mat = bpy.data.materials.new(name=nombre)
    mat.diffuse_color = (*color_rgb, 1.0)
    return mat


def generar_pasillo_curva_suave():
    """
    Genera un pasillo con curvas suaves continuas y cámara que recorre todo el trayecto
    """
    
    # ========== PASO 1: Limpieza del Entorno ==========
    bpy.ops.object.select_all(action='SELECT')
    bpy.ops.object.delete()
    
    
    # ========== PASO 2: Definición de Materiales ==========
    mat_pared_oscura = crear_material("ParedOscura", (0.1, 0.1, 0.1))
    mat_pared_naranja = crear_material("ParedNaranja", (0.8, 0.3, 0.1))
    mat_suelo_blanco = crear_material("SueloBlanco", (1.0, 1.0, 1.0))
    
    
    # ========== PASO 3: Parámetros del Pasillo ==========
    ancho_pasillo = 3.5          # Ancho entre las dos líneas de bloques
    num_bloques = 40             # Total de bloques a lo largo del camino
    longitud_total = 50          # Longitud total del recorrido
    amplitud_curva = 8           # Cuánto se curva (mayor = más ondulado)
    frecuencia_curva = 2         # Número de ondulaciones
    
    
    # ========== PASO 4: Generar Puntos del Recorrido (Curva Suave) ==========
    posiciones_camino = []
    
    for i in range(num_bloques):
        # Progreso a lo largo del camino (0 a 1)
        t = i / (num_bloques - 1)
        
        # Posición en Y (avanza a lo largo del camino)
        y = t * longitud_total
        
        # Posición en X (crea la curva suave usando función seno)
        x = amplitud_curva * math.sin(frecuencia_curva * math.pi * t)
        
        # Calcular ángulo de rotación basado en la tangente de la curva
        dx_dt = amplitud_curva * frecuencia_curva * math.pi * math.cos(frecuencia_curva * math.pi * t)
        dy_dt = longitud_total
        angulo_rotacion = math.atan2(dx_dt, dy_dt)
        
        posiciones_camino.append({
            'x': x,
            'y': y,
            'angulo': angulo_rotacion
        })
    
    
    # ========== PASO 5: Crear Bloques del Pasillo ==========
    for i, pos in enumerate(posiciones_camino):
        x_centro = pos['x']
        y_centro = pos['y']
        angulo = pos['angulo']
        
        # Calcular vector perpendicular para posicionar bloques a los lados
        dx = math.cos(angulo)
        dy = -math.sin(angulo)
        
        # ===== BLOQUE IZQUIERDO =====
        x_izq = x_centro - dx * (ancho_pasillo / 2)
        y_izq = y_centro - dy * (ancho_pasillo / 2)
        
        bpy.ops.mesh.primitive_cube_add(location=(x_izq, y_izq, 0.5))
        bloque_izq = bpy.context.active_object
        bloque_izq.rotation_euler.z = angulo
        bloque_izq.scale = (0.8, 0.8, 1.0)
        
        # Alternar colores
        if i % 2 == 0:
            bloque_izq.data.materials.append(mat_pared_oscura)
        else:
            bloque_izq.data.materials.append(mat_pared_naranja)
        
        
        # ===== BLOQUE DERECHO =====
        x_der = x_centro + dx * (ancho_pasillo / 2)
        y_der = y_centro + dy * (ancho_pasillo / 2)
        
        bpy.ops.mesh.primitive_cube_add(location=(x_der, y_der, 0.5))
        bloque_der = bpy.context.active_object
        bloque_der.rotation_euler.z = angulo
        bloque_der.scale = (0.8, 0.8, 1.0)
        
        # Alternar colores (opuesto al izquierdo)
        if i % 2 == 0:
            bloque_der.data.materials.append(mat_pared_naranja)
        else:
            bloque_der.data.materials.append(mat_pared_oscura)
        
        
        # ===== SUELO =====
        bpy.ops.mesh.primitive_plane_add(size=1.5, location=(x_centro, y_centro, 0))
        suelo = bpy.context.active_object
        suelo.rotation_euler.z = angulo
        suelo.scale.x = ancho_pasillo / 1.5 + 0.3
        suelo.scale.y = 1.0
        suelo.data.materials.append(mat_suelo_blanco)
    
    
    # ========== PASO 6: Crear Curva Bezier Suave para la Cámara ==========
    # Crear más puntos para una curva más suave
    num_puntos_camara = num_bloques * 2  # Más puntos = movimiento más suave
    
    bpy.ops.curve.primitive_bezier_curve_add(location=(0, 0, 0))
    path_curve = bpy.context.active_object
    path_curve.name = "CameraPath"
    
    # Limpiar puntos por defecto
    curve_data = path_curve.data
    curve_data.splines.clear()
    
    # Crear nueva spline tipo BEZIER (más suave que POLY)
    spline = curve_data.splines.new('BEZIER')
    spline.bezier_points.add(num_puntos_camara - 1)
    
    # Generar puntos de la cámara con más densidad
    for i in range(num_puntos_camara):
        t = i / (num_puntos_camara - 1)
        y = t * longitud_total
        x = amplitud_curva * math.sin(frecuencia_curva * math.pi * t)
        
        # Altura de la cámara
        altura_camara = 2.0
        
        # Configurar punto bezier
        point = spline.bezier_points[i]
        point.co = (x, y, altura_camara)
        point.handle_left_type = 'AUTO'
        point.handle_right_type = 'AUTO'
    
    # Configurar la curva para que sea suave
    curve_data.dimensions = '3D'
    curve_data.resolution_u = 24  # Alta resolución para suavidad
    
    
    # ========== PASO 7: Crear Cámara con Follow Path SIMPLIFICADO ==========
    pos_inicial = posiciones_camino[0]
    
    # Crear cámara
    bpy.ops.object.camera_add(location=(pos_inicial['x'], pos_inicial['y'], 2.0))
    camera = bpy.context.active_object
    camera.name = "CameraSeguimiento"
    
    # Configurar cámara
    camera.data.lens = 35
    camera.data.clip_start = 0.1
    camera.data.clip_end = 1000
    
    # Hacer que esta sea la cámara activa
    bpy.context.scene.camera = camera
    
    # Agregar constraint "Follow Path" - ESTE ES EL CLAVE
    follow_path = camera.constraints.new(type='FOLLOW_PATH')
    follow_path.target = path_curve
    follow_path.use_curve_follow = True  # Esto hace que siga la orientación de la curva
    follow_path.use_fixed_location = False
    follow_path.forward_axis = 'FORWARD_Y'  # La cámara apunta hacia adelante (eje Y)
    follow_path.up_axis = 'UP_Z'  # El eje Z apunta hacia arriba
    
    
    # ========== PASO 8: Animar la Cámara a lo Largo de la Curva ==========
    num_frames = 300  # Duración de la animación (300 frames = 12.5 segundos a 24fps)
    bpy.context.scene.frame_start = 1
    bpy.context.scene.frame_end = num_frames
    
    # Configurar FPS para mejor control
    bpy.context.scene.render.fps = 24
    
    # Keyframe al inicio (offset = 0 significa inicio de la curva)
    follow_path.offset = -0
    follow_path.keyframe_insert(data_path="offset", frame=1)
    
    # Keyframe al final (offset = 100 significa final de la curva)
    follow_path.offset = -100
    follow_path.keyframe_insert(data_path="offset", frame=num_frames)
    
    # Hacer la interpolación lineal (velocidad constante)
    if camera.animation_data and camera.animation_data.action:
        for fcurve in camera.animation_data.action.fcurves:
            for keyframe in fcurve.keyframe_points:
                keyframe.interpolation = 'LINEAR'
    
    
    # ========== PASO 9: Iluminación ==========
    # Luz solar general
    bpy.ops.object.light_add(type='SUN', location=(0, 25, 30))
    sol = bpy.context.active_object
    sol.data.energy = 1.5
    sol.rotation_euler = (math.radians(50), 0, math.radians(20))
    
    # Luces puntuales a lo largo del recorrido
    for i in range(0, len(posiciones_camino), 5):
        pos = posiciones_camino[i]
        bpy.ops.object.light_add(type='POINT', location=(pos['x'], pos['y'], 4))
        luz = bpy.context.active_object
        luz.data.energy = 500
        luz.data.color = (1.0, 0.95, 0.85)
    
    
    # ========== PASO 10: Ocultar la Curva Path del Render ==========
    path_curve.hide_render = True
    path_curve.hide_viewport = True  # También ocultarla en el viewport
    
    
    print("=" * 70)
    print("✓ PASILLO CON CURVAS SUAVES Y CÁMARA EN SEGUIMIENTO GENERADO!")
    print("=" * 70)
    print(f"📊 Bloques por lado: {num_bloques}")
    print(f"🎬 Frames de animación: {num_frames} ({num_frames/24:.1f} segundos)")
    print(f"📏 Longitud del recorrido: {longitud_total} unidades")
    print(f"🌊 Amplitud de curva: {amplitud_curva}")
    print(f"📹 Puntos de cámara: {num_puntos_camara}")
    print()
    print("🎮 CÓMO VER LA ANIMACIÓN:")
    print("  1. Presiona NUMPAD 0 (vista desde cámara)")
    print("  2. Presiona Z → Material Preview o Rendered")
    print("  3. Presiona ESPACIO (reproducir animación)")
    print("  4. La cámara recorrerá TODA la curva automáticamente")
    print()
    print("💡 TIP: Si la cámara va muy rápida o lenta, cambia 'num_frames'")
    print("=" * 70)


# ========== EJECUTAR ==========
if __name__ == "__main__":
    generar_pasillo_curva_suave()
    <img width="846" height="488" alt="image" src="https://github.com/user-attachments/assets/423c0b91-0ee1-4fd7-97bb-3de672ff4363" />
    <img width="2940" height="1912" alt="image" src="https://github.com/user-attachments/assets/0f0b5670-0e9c-4882-81b6-667d42a12314" />

