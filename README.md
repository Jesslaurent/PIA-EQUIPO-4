# PIA-EQUIPO-4
import matplotlib.pyplot as plt
import numpy as np
import csv

class Alumno:
    def __init__(self, matricula, nombre, apellido_p, apellido_m, semestre):
        self.matricula = matricula
        self.nombre = f"{nombre} {apellido_p} {apellido_m}"
        self.semestre = semestre
        self.calificaciones = [0, 0, 0, 0, 0]  # 3 exámenes, act, PIA

    def calcular_promedio(self):
        ponderacion = [0.15, 0.15, 0.15, 0.20, 0.35]
        promedio = sum(p * c for p, c in zip(ponderacion, self.calificaciones))
        return promedio


class Grupo:
    def __init__(self, numero_grupo, aula, materia, max_alumnos):
        # Validar que el grupo tenga formato "número + letra"
        if not numero_grupo[:-1].isdigit() or not numero_grupo[-1].isalpha():
            raise ValueError("Formato de grupo incorrecto. Debe ser número seguido de letra.")
        
        # Validar que el aula sea un número
        if not aula.isdigit():
            raise ValueError("El número de aula debe ser un valor numérico.")
        
        self.numero_grupo = numero_grupo
        self.aula = aula
        self.materia = materia
        self.max_alumnos = max_alumnos
        self.alumnos = []
        self.matriculas_registradas = set()  # Conjunto de matrículas ya registradas

    def registrar_alumno(self, alumno):
        if alumno.matricula in self.matriculas_registradas:
            print("¡Error! Esta matrícula ya está registrada en el grupo.")
            return False

        if len(self.alumnos) < self.max_alumnos:
            self.alumnos.append(alumno)
            self.matriculas_registradas.add(alumno.matricula)  # Agregar matrícula al conjunto
            return True
        else:
            return False

    def calcular_promedio_grupo(self):
        promedios = [alumno.calcular_promedio() for alumno in self.alumnos]
        promedio_grupo = sum(promedios) / len(promedios) if len(promedios) > 0 else 0
        return promedio_grupo


def guardar_datos_en_archivo():
    for grupo in grupos:
        nombre_archivo = f"Grupo{grupo.numero_grupo}.csv"
        with open(nombre_archivo, "w", newline='') as archivo:
            writer = csv.writer(archivo)

            encabezados = ["Nombre del alumno", "Matrícula", "Aula", "Semestre", "Grupo", "Examen 1", "Examen 2", "Examen 3", "Actividades", "PIA", "Promedio"]
            writer.writerow(encabezados)

            for alumno in grupo.alumnos:
                datos = [
                    alumno.nombre, alumno.matricula, grupo.aula, str(alumno.semestre), str(grupo.numero_grupo),
                    "{:.2f}".format(alumno.calificaciones[0]),  # Examen 1
                    "{:.2f}".format(alumno.calificaciones[1]),  # Examen 2
                    "{:.2f}".format(alumno.calificaciones[2]),  # Examen 3
                    "{:.2f}".format(alumno.calificaciones[3]),  # Actividades
                    "{:.2f}".format(alumno.calificaciones[4]),  # PIA
                    "{:.2f}".format(alumno.calcular_promedio())  # Promedio
                ]
                writer.writerow(datos)

def graficar_promedios(grupos):
    nombres_grupos = [f"Grupo {grupo.numero_grupo}" for grupo in grupos]
    promedios = [grupo.calcular_promedio_grupo() for grupo in grupos]

    # Generar colores automáticamente
    colores = plt.cm.viridis(np.linspace(0, 1, len(nombres_grupos)))

    # Crear la gráfica de barras
    plt.bar(nombres_grupos, promedios, color=colores)
    plt.xlabel('Grupos')
    plt.ylabel('Promedio')
    plt.title('Promedio de Grupos')
    plt.xticks(rotation=45, ha='right')

    # Mostrar la gráfica
    plt.show()               


grupos = []

try:
    with open("Exposicion.txt", "r") as file:
        grupo = None
        for line in file:
            parts = line.strip().split(",")
            if parts[0] == "Grupo":
                numero_grupo, aula, materia, max_alumnos = parts[1:]
                # Añadir validación de formato de grupo y aula
                grupo = Grupo(numero_grupo, aula, materia, int(max_alumnos))
                grupos.append(grupo)
            elif parts[0] == "Alumno" and grupo is not None:
                matricula, nombre, apellido_p, apellido_m, semestre = parts[1:6]
                # Validar nombre del alumno
                if not nombre.isalpha() or not apellido_p.isalpha() or not apellido_m.isalpha() or len(nombre) < 7:
                    raise ValueError("Nombre de estudiante inválido.")
                # Validar edad del estudiante
                alumno = Alumno(matricula, nombre, apellido_p, apellido_m, semestre)
                calificaciones = [float(calif) for calif in parts[6:]]
                # Validar calificaciones
                if not all(0 <= calificacion <= 100 for calificacion in calificaciones):
                    raise ValueError("Las calificaciones deben estar entre 0 y 100.")
                alumno.calificaciones = calificaciones
                grupo.registrar_alumno(alumno)
except FileNotFoundError:
    pass

while True:
    print("\nMenú principal:")
    print("1. Registrar nuevo Grupo")
    print("2. Registrar nuevo alumno")
    print("3. Registrar calificaciones")
    print("4. Consultar promedio final por matrícula de Alumno")
    print("5. Consultar promedio de Grupo por No. Grupo")
    print("6. Consultar Grupo con mejor promedio")
    print("7. Buscar Alumno por nombre o apellidos")
    print("8. Salir")
    opcion = input("Seleccione una opción: ")

    if opcion == "1":
        while True:
            numero_grupo = input("No. Grupo: ")
            
            # Validar que el grupo tenga formato "número + letra"
            if numero_grupo[:-1].isdigit() and numero_grupo[-1].isalpha():
                break
            else:
                print("Formato de grupo incorrecto. Debe ser número seguido de letra. Intente de nuevo.")

        aula = input("Aula: ")
        materia = input("Materia: ")
        max_alumnos = int(input("Cantidad de Alumnos: "))

        # Ya que se validó el formato de número de grupo, se crea el grupo
        grupo = Grupo(numero_grupo, aula, materia, max_alumnos)
        grupos.append(grupo)
        print("Grupo registrado con éxito.")

    elif opcion == "2":

        while True:
            nombre = input("Nombre: ")
            if nombre.replace(" ", "").isalpha():  # Permitir espacios en el nombre
                break
            else:
                print("Error: El nombre solo debe contener letras. Intente de nuevo.")

        while True:
            apellido_p = input("Apellido Paterno: ")
            if apellido_p.isalpha():
                break
            else:
                print("Error: El apellido paterno solo debe contener letras. Intente de nuevo.")

        while True:
            apellido_m = input("Apellido Materno: ")
            if apellido_m.isalpha():
                break
            else:
                print("Error: El apellido materno solo debe contener letras. Intente de nuevo.")

        while True:
            matricula = input("Matrícula: ")
            if matricula.isdigit():
                break
            else:
                print("Error: La matrícula debe contener solo números. Intente de nuevo.")

        while True:
            semestre = input("Semestre: ")
            if semestre.isdigit():
                semestre = int(semestre)  # Convertir a entero
                break
            else:
                print("Error: El semestre debe ser un número. Intente de nuevo.")

        while True:
            numero_grupo = input("Número de Grupo: ")
            if numero_grupo[:-1].isdigit() and numero_grupo[-1].isalpha():
                break
            else:
                print("Error: El número de grupo debe ser números seguidos de una letra. Intente de nuevo.")

        # Verificar si el grupo existe
        grupo_encontrado = next((grupo for grupo in grupos if grupo.numero_grupo == numero_grupo), None)
        if grupo_encontrado:
            alumno = Alumno(matricula, nombre, apellido_p, apellido_m, semestre)
            if grupo_encontrado.registrar_alumno(alumno):
                print("Alumno registrado en el grupo con éxito.")
            else:
                print("No hay espacio en el grupo seleccionado para este alumno.")
        else:
            print("Grupo no encontrado. Asegúrate de registrar el grupo primero.")


    elif opcion == "3":
        matricula = input("Matrícula del alumno: ")
        alumno_encontrado = False

        for grupo in grupos:
            for alumno in grupo.alumnos:
                if alumno.matricula == matricula:
                    print(f"Calificaciones del alumno {alumno.nombre}:")
                    # Solicitar las calificaciones hasta que estén dentro del rango
                    while True:
                        try:
                            examen1 = float(input("Examen 1: "))
                            if 0 <= examen1 <= 100:
                                break
                            else:
                                print("¡Error! La calificación debe estar entre 0 y 100.")
                        except ValueError:
                            print("¡Error! Ingrese un número válido.")
                        
                    while True:
                        try:
                            examen2 = float(input("Examen 2: "))
                            if 0 <= examen2 <= 100:
                                break
                            else:
                                print("¡Error! La calificación debe estar entre 0 y 100.")
                        except ValueError:
                            print("¡Error! Ingrese un número válido.")
                    
                    while True:
                        try:
                            examen3 = float(input("Examen 3: "))
                            if 0 <= examen3 <= 100:
                                break
                            else:
                                print("¡Error! La calificación debe estar entre 0 y 100.")
                        except ValueError:
                            print("¡Error! Ingrese un número válido.")
                    
                    while True:
                        try:
                            actividades = float(input("Actividades: "))
                            if 0 <= actividades <= 100:
                                break
                            else:
                                print("¡Error! La calificación debe estar entre 0 y 100.")
                        except ValueError:
                            print("¡Error! Ingrese un número válido.")
                    
                    while True:
                        try:
                            pia = float(input("PIA: "))
                            if 0 <= pia <= 100:
                                break
                            else:
                                print("¡Error! La calificación debe estar entre 0 y 100.")
                        except ValueError:
                            print("¡Error! Ingrese un número válido.")

                    alumno.calificaciones = [examen1, examen2, examen3, actividades, pia]
                    alumno_encontrado = True
                    break

        if not alumno_encontrado:
            print("Matrícula de alumno no encontrada.")
    elif opcion == "4":
        matricula = input("Matrícula del alumno: ")
        alumno_encontrado = False

        for grupo in grupos:
           for alumno in grupo.alumnos: 
                if alumno.matricula == matricula:
                    print(f"Nombre del alumno: {alumno.nombre}")
                    promedio = alumno.calcular_promedio()
                    print(f"Promedio: {promedio}")
                    alumno_encontrado = True
                    break
        if not alumno_encontrado:
            print("Matrícula de alumno no encontrada.")
    elif opcion == "5":
        numero_grupo = input("No. Grupo: ")
        grupo_encontrado = False

        for grupo in grupos:
            if grupo.numero_grupo == numero_grupo:
                promedio_grupo = grupo.calcular_promedio_grupo()
                print(f"Promedio del grupo {grupo.numero_grupo}: {promedio_grupo}")
                grupo_encontrado = True
                break
        if not grupo_encontrado:
            print("No se encontró el grupo.")
    elif opcion == "6":
        grupo_mejor_promedio = None
        mejor_promedio = 0

        for grupo in grupos:
            promedio_grupo = grupo.calcular_promedio_grupo()
            if promedio_grupo > mejor_promedio:
                grupo_mejor_promedio = grupo
                mejor_promedio = promedio_grupo

        if grupo_mejor_promedio is not None:
            print(f"El grupo con mejor promedio es el grupo {grupo_mejor_promedio.numero_grupo} con un promedio de {mejor_promedio}.")
            # Llamar a la función para graficar los promedios
            graficar_promedios(grupos)
        else:
            print("No hay grupos registrados.")

    elif opcion == "7":
        busqueda = input("Introduce el nombre o apellidos del alumno a buscar: ")
        encontrado = False

        for grupo in grupos:
            for alumno in grupo.alumnos:
                if busqueda in alumno.nombre or busqueda in alumno.matricula:
                    print(f"Alumno: {alumno.nombre}, Matrícula: {alumno.matricula}")
                    print(f"Semestre: {alumno.semestre}")
                    print(f"Promedio: {alumno.calcular_promedio()}")
                    encontrado = True
        if not encontrado:
            print("No se encontraron coincidencias.")

    elif opcion == "8":
        print("Saliendo del programa.")
        guardar_datos_en_archivo()
        break
    else:
        print("Opción no válida. Introduce una opción válida del menú.")

    guardar_datos_en_archivo()
