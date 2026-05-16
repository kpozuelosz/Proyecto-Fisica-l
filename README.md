import numpy as np
import matplotlib.pyplot as plt
import matplotlib.animation as animation
from matplotlib.offsetbox import OffsetImage, AnnotationBbox

pixeles_pesa1 = np.array([
    [0, 1, 1, 0],
    [1, 1, 1, 1],
    [1, 1, 1, 1],
    [0, 1, 1, 0]
])

pixeles_pesa2 = np.array([
    [0, 0, 1, 1, 0, 0],
    [0, 1, 1, 1, 1, 0],
    [1, 1, 1, 1, 1, 1],
    [1, 1, 1, 1, 1, 1],
    [0, 1, 1, 1, 1, 0],
    [0, 0, 1, 1, 0, 0]
])

def crear_imagen_pixel(matriz_pixeles):
    """Convierte una matriz de píxeles en una imagen RGBA para Matplotlib."""
    h, w = matriz_pixeles.shape
    imagen_rgba = np.zeros((h, w, 4))
    imagen_rgba[:, :, :3] = 0  # Color negro para los píxeles con valor 1
    imagen_rgba[:, :, 3] = matriz_pixeles  # Canal Alfa (transparencia) es la matriz
    return imagen_rgba

imagen_p1 = crear_imagen_pixel(pixeles_pesa1)
imagen_p2 = crear_imagen_pixel(pixeles_pesa2)

def mostrar_menu():
    print("\n" + "="*40)
    print("      --- SIMULADOR DE LEYES DE NEWTON ---      ")
    print("="*40)
    print("1. Segunda Ley: Aceleración de un bloque con fricción")
    print("   (La fuerza aplicada actúa como tensión en el cable)")
    print("2. Tercera Ley: Tensión en sistema de dos masas (Atwood)")
    print("   (Un bloque en la mesa conectado a una masa colgante)")
    print("="*40)

def configurar_segunda_ley():
    print("\n" + "-"*30)
    print("   --- CONFIGURACIÓN 2DA LEY ---   ")
    print("-"*30)
    try:
        m = float(input("Masa del bloque m1 (kg): "))
        f_aplicada = float(input("Fuerza aplicada / Tensión del cable (N): "))
        mu = float(input("Coeficiente de fricción cinética (0 para nula): "))
    except ValueError:
        print("Error: Ingrese valores numéricos válidos.")
        return None, None, None, None

    g = 9.81
    f_roce = mu * m * g
    
    if f_aplicada > f_roce:
        f_total = f_aplicada - f_roce
        accel = f_total / m
    else:
        accel = 0
        print("\n[Aviso]: La fuerza aplicada no supera la fuerza de fricción estática/cinética. El bloque no se moverá.")
        
    tension = f_aplicada 

    print("\n" + "-"*30)
    print(f"      --- RESULTADOS ---      ")
    print("-"*30)
    print(f"Fuerza de Fricción: {f_roce:.2f} N")
    print(f"Aceleración: {accel:.2f} m/s^2")
    print(f"Tensión dinámica: {tension:.2f} N")
    print("-"*30)
    return accel, tension, m, 1 # Se envía una masa ficticia 1 para activar el diseño visual

def configurar_tercera_ley():
    print("\n" + "-"*30)
    print("   --- CONFIGURACIÓN 3RA LEY (Sistema) ---   ")
    print("-"*30)
    try:
        m1 = float(input("Masa sobre la mesa m1 (kg): "))
        m2 = float(input("Masa colgante m2 (kg): "))
        mu = float(input("Coeficiente de fricción en la mesa (cinética): "))
    except ValueError:
        print("Error: Ingrese valores numéricos válidos.")
        return None, None, None, None

    g = 9.81
    f_roce = mu * m1 * g
    
    # El sistema solo se mueve si el peso de m2 es mayor que la fricción de m1
    if (m2 * g) > f_roce:
        accel = (m2 * g - f_roce) / (m1 + m2)
        tension = m1 * accel + f_roce
    else:
        accel = 0
        tension = m2 * g # Si no se mueve, la tensión equilibra el peso de m2
        print("\n[Aviso]: El peso de la masa colgante no supera la fricción. El sistema permanece estático.")

    print("\n" + "-"*30)
    print(f"      --- RESULTADOS ---      ")
    print("-"*30)
    print(f"Fuerza de Fricción: {f_roce:.2f} N")
    print(f"Aceleración: {accel:.2f} m/s^2")
    print(f"Tensión en el cable: {tension:.2f} N")
    print("-"*30)
    return accel, tension, m1, m2

def animar_sistema(accel, tension, m1, m2):
    if accel is None: return

    # Configuración de la figura
    fig, ax = plt.subplots(figsize=(9, 6), facecolor='whitesmoke')
    ax.set_xlim(-1, 6)
    ax.set_ylim(-4, 3)
    ax.set_aspect('equal')
    ax.grid(True, linestyle=':', color='silver')

    # Decoración: Mesa y Polea
    mesa_borde, = ax.plot([0, 4, 4], [0, 0, -0.1], 'k', lw=4, solid_capstyle='round', label='Borde de la Mesa')
    mesa_cuerpo, = ax.plot([0, 4], [-0.05, -0.05], 'tan', lw=10, solid_capstyle='butt') 
    ax.plot([4], [0], 'ko', markersize=8, markeredgewidth=2) # Punto de la polea

    cuerda, = ax.plot([], [], 'dimgray', lw=2.5, alpha=0.8, label='Cuerda / Cable')

    im_p1 = OffsetImage(imagen_p1, zoom=4) 
    im_p2 = OffsetImage(imagen_p2, zoom=4)

    box_p1 = AnnotationBbox(im_p1, (0, 0), frameon=False)
    box_p2 = AnnotationBbox(im_p2, (0, 0), frameon=False)
    ax.add_artist(box_p1)
    ax.add_artist(box_p2)

    txt = ax.text(0.3, 1.8, f"Resultados de la Simulación:\nAceleración: {accel:.2f} m/s²\nTensión: {tension:.2f} N", 
                  fontsize=12, family='serif', color='navy',
                  bbox={'facecolor':'white', 'edgecolor':'silver', 'alpha':0.8, 'pad':8})

    ax.legend(loc='lower left')
    plt.title("Simulación Dinámica: Bloque y Pesa", fontsize=16, family='serif', weight='bold')

    def update(frame):
        t = frame / 20
        d = 0.5 * accel * (t**2)
        
        # Posición del bloque m1 en la mesa
        pos_b1_x = min(2 + d, 4)
        pos_b1_y = 0.4 
        
        # Posición del extremo de la tensión / peso colgante m2
        pos_b2_x = 4.3 
        pos_b2_y = -0.6 - (pos_b1_x - 2) 
        
        box_p1.xybox = (pos_b1_x, pos_b1_y)
        box_p2.xybox = (pos_b2_x, pos_b2_y)

        cuerda.set_data([pos_b1_x, 4.05, 4.05], [pos_b1_y, pos_b1_y, pos_b2_y + 0.3]) 
        
        return cuerda, box_p1, box_p2

    ani = animation.FuncAnimation(fig, update, frames=150, interval=60, blit=True)
    plt.show()

def main():
    while True:
        mostrar_menu()
        opcion = input("Seleccione una opción (1 o 2, o 's' para salir): ").strip().lower()

        if opcion == '1':
            accel, tension, m1, m2 = configurar_segunda_ley()
            if accel is not None: animar_sistema(accel, tension, m1, m2)
        elif opcion == '2':
            accel, tension, m1, m2 = configurar_tercera_ley()
            if accel is not None: animar_sistema(accel, tension, m1, m2)
        elif opcion == 's':
            print("\n¡Gracias por usar el simulador! ¡Adiós!")
            break
        else:
            print("\nError: Opción no válida. Intente de nuevo.")

if __name__ == "__main__":
    main()
