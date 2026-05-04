# Viky.santanilla-proyecto-4
Desarrollo Fase 4 de curso programacion
from abc import ABC, abstractmethod
import datetime

# =========================
# LOGS
# =========================
def registro(mensaje):
    with open("logs.txt", "a", encoding="utf-8") as f:
        f.write(f"{datetime.datetime.now()} - {mensaje}\n")

# =========================
# EXCEPCIONES
# =========================
class ReservaError(Exception):
    pass

class ClienteError(Exception):
    pass

# =========================
# CLASE CLIENTE
# =========================
class Cliente:
    def __init__(self, nombre, identificacion):
        if not nombre.strip():
            raise ClienteError("Nombre vacío")
        if not identificacion.strip():
            raise ClienteError("Identificación vacía")

        self.__nombre = nombre
        self.__identificacion = identificacion

    def obtener_nombre(self):
        return self.__nombre

# =========================
# CLASE ABSTRACTA SERVICIO
# =========================
class Servicio(ABC):
    def __init__(self, nombre, precio_base):
        self.nombre = nombre
        self.precio_base = precio_base

    @abstractmethod
    def calcular_costo(self, tiempo):
        pass

    @abstractmethod
    def descripcion(self):
        pass

# =========================
# SERVICIOS
# =========================
class ReservaSala(Servicio):
    def calcular_costo(self, horas):
        if horas <= 0:
            raise ReservaError("Horas inválidas")
        return self.precio_base * horas

    def descripcion(self):
        return "Reserva de sala"

class AlquilerEquipo(Servicio):
    def calcular_costo(self, dias):
        if dias <= 0:
            raise ReservaError("Días inválidos")
        return self.precio_base * dias

    def descripcion(self):
        return "Alquiler de equipo"

class Asesoria(Servicio):
    def calcular_costo(self, horas, descuento=0):
        if horas <= 0:
            raise ReservaError("Horas inválidas")
        costo = self.precio_base * horas
        return costo - (costo * descuento)

    def descripcion(self):
        return "Asesoría especializada"

# =========================
# CLASE RESERVA
# =========================
class Reserva:
    def __init__(self, cliente, servicio, duracion):
        if not isinstance(cliente, Cliente):
            raise ReservaError("Cliente inválido")
        if not isinstance(servicio, Servicio):
            raise ReservaError("Servicio inválido")
        if duracion <= 0:
            raise ReservaError("Duración inválida")

        self.cliente = cliente
        self.servicio = servicio
        self.duracion = duracion
        self.estado = "Pendiente"

    def confirmar(self):
        self.estado = "Confirmada"
        registro("Reserva confirmada")

    def cancelar(self):
        self.estado = "Cancelada"
        registro("Reserva cancelada")

    def procesar(self):
        costo = self.servicio.calcular_costo(self.duracion)
        registro(f"Reserva procesada - Costo: {costo}")
        return costo

    def mostrar(self):
        return f"{self.cliente.obtener_nombre()} - {self.servicio.descripcion()} - {self.estado}"

# =========================
# SIMULACIÓN (10+ operaciones)
# =========================
def simulacion():
    clientes = []
    servicios = []
    reservas = []

    try:
        c1 = Cliente("Ana", "123")
        clientes.append(c1)
    except Exception as e:
        registro(e)

    try:
        c2 = Cliente("", "456")  # ERROR
        clientes.append(c2)
    except Exception as e:
        registro(e)

    try:
        s1 = ReservaSala("Sala VIP", 50000)
        s2 = AlquilerEquipo("Proyector", 30000)
        s3 = Asesoria("Consultoría", 80000)
        servicios.extend([s1, s2, s3])
    except Exception as e:
        registro(e)

    try:
        r1 = Reserva(c1, s1, 2)
        r1.procesar()
        r1.confirmar()
        reservas.append(r1)
    except Exception as e:
        registro(e)

    try:
        r2 = Reserva(c1, s2, -1)  # ERROR
        r2.procesar()
        reservas.append(r2)
    except Exception as e:
        registro(e)

    try:
        r3 = Reserva(c1, s3, 3)
        r3.procesar()
        reservas.append(r3)
    except Exception as e:
        registro(e)

    print("Simulación finalizada. Revisa logs.txt")

# =========================
# MENÚ INTERACTIVO
# =========================
def menu():
    clientes = []
    servicios = []
    reservas = []

    while True:
        print("\n=== SOFTWARE FJ ===")
        print("1. Crear cliente")
        print("2. Crear servicio")
        print("3. Crear reserva")
        print("4. Ver reservas")
        print("5. Cancelar reserva")
        print("6. Salir")

        opcion = input("Seleccione: ")

        try:
            if opcion == "1":
                nombre = input("Nombre: ")
                identificacion = input("ID: ")
                cliente = Cliente(nombre, identificacion)
                clientes.append(cliente)
                print("Cliente creado")

            elif opcion == "2":
                print("1. Sala 2. Equipo 3. Asesoría")
                tipo = input("Seleccione: ")

                if tipo == "1":
                    servicio = ReservaSala("Sala", 50000)
                elif tipo == "2":
                    servicio = AlquilerEquipo("Equipo", 30000)
                elif tipo == "3":
                    servicio = Asesoria("Asesoría", 80000)
                else:
                    print("Opción inválida")
                    continue

                servicios.append(servicio)
                print("Servicio creado")

            elif opcion == "3":
                if not clientes or not servicios:
                    print("Debe crear clientes y servicios primero")
                    continue

                for i, c in enumerate(clientes):
                    print(f"{i} - {c.obtener_nombre()}")
                idx_c = int(input("Cliente: "))

                for i, s in enumerate(servicios):
                    print(f"{i} - {s.nombre}")
                idx_s = int(input("Servicio: "))

                duracion = int(input("Duración: "))

                reserva = Reserva(clientes[idx_c], servicios[idx_s], duracion)
                reserva.procesar()
                reserva.confirmar()
                reservas.append(reserva)

                print("Reserva creada")

            elif opcion == "4":
                for r in reservas:
                    print(r.mostrar())

            elif opcion == "5":
                for i, r in enumerate(reservas):
                    print(f"{i} - {r.mostrar()}")
                idx = int(input("Seleccione: "))
                reservas[idx].cancelar()

            elif opcion == "6":
                print("Saliendo...")
                break

            else:
                print("Opción inválida")

        except Exception as e:
            print("Error:", e)
            registro(str(e))

# =========================
# EJECUCIÓN
# =========================
if __name__ == "__main__":
    simulacion()
    menu()