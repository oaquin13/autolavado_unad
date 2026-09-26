"""
UNAD - Universidad Nacional Abierta y a Distancia
Escuela de Ciencias Básicas, Tecnología e Ingeniería (ECBTI)
Programa: Ingeniería de Sistemas
Curso: Programación (Código: 213023)
Fase 2 - Ejercicio 3: Sistema para Control de Lavado
Lenguaje: Python 3
GUI: Tkinter
"""

import math
import tkinter as tk
from tkinter import messagebox, ttk


# ==============================================================================
# MODELO DE DATOS (POO / ENCAPSULAMIENTO)
# ==============================================================================


class Usuario:
    """Clase que gestiona la autenticación del usuario del sistema."""

    def __init__(self):
        # Atributos privados requeridos por la guía
        self._usuario = "programacion"
        self._password = "programacion"

    def validar(self, usuario_ingresado: str, password_ingresada: str) -> bool:
        """Valida credenciales contra atributos privados."""
        return (
            self._usuario == usuario_ingresado
            and self._password == password_ingresada
        )


class AutoLavado:
    """Clase que representa un vehículo dentro del sistema de autolavado."""

    def __init__(self, placa: str, tarifa_hora: float):
        # Atributos privados requeridos por la guía
        self._placa = placa.upper().strip()
        self._hora_ingreso = 0.0
        self._hora_salida = 0.0
        self._tarifa_hora = tarifa_hora

    def registrar_ingreso(self, hora: float) -> None:
        """Registra la hora de ingreso del vehículo."""
        self._hora_ingreso = hora

    def registrar_salida(self, hora: float) -> None:
        """Registra la hora de salida del vehículo."""
        self._hora_salida = hora

    def calcular_pago(self, hora_salida: float) -> float:
        """Calcula el costo total cobrado por horas o fracción de hora."""
        tiempo_transcurrido = hora_salida - self._hora_ingreso
        # Se redondea hacia arriba las horas o fracciones de hora
        horas_cobradas = math.ceil(tiempo_transcurrido)
        return horas_cobradas * self._tarifa_hora

    def obtener_placa(self) -> str:
        """Retorna la placa del vehículo para identificación."""
        return self._placa


# ==============================================================================
# INTERFAZ GRÁFICA EN TKINTER (GUI EN INGLÉS)
# ==============================================================================


class LoginWindow:
    """Ventana de inicio de sesión."""

    def __init__(self, root: tk.Tk):
        self.root = root
        self.root.title("System Authentication - UNAD")
        self.root.geometry("400x300")
        self.root.resizable(False, False)

        self.usuario_model = Usuario()

        # Estilos de la UI
        frame = ttk.Frame(self.root, padding=30)
        frame.pack(expand=True, fill="both")

        ttk.Label(
            frame,
            text="Car Wash System Login",
            font=("Helvetica", 14, "bold"),
        ).pack(pady=(0, 20))

        # Input Usuario
        ttk.Label(frame, text="Username:", font=("Helvetica", 10)).pack(
            anchor="w"
        )
        self.txt_user = ttk.Entry(frame, font=("Helvetica", 10))
        self.txt_user.pack(fill="x", pady=(2, 10))
        self.txt_user.focus()

        # Input Password
        ttk.Label(frame, text="Password:", font=("Helvetica", 10)).pack(
            anchor="w"
        )
        self.txt_pass = ttk.Entry(frame, font=("Helvetica", 10), show="*")
        self.txt_pass.pack(fill="x", pady=(2, 20))

        # Botón Login
        btn_login = ttk.Button(
            frame, text="Log In", command=self.autenticar_usuario
        )
        btn_login.pack(fill="x")

        # Permitir iniciar con tecla Enter
        self.root.bind("<Return>", lambda event: self.autenticar_usuario())

    def autenticar_usuario(self):
        usr = self.txt_user.get().strip()
        pwd = self.txt_pass.get().strip()

        if self.usuario_model.validar(usr, pwd):
            messagebox.showinfo(
                "Access Granted", "Welcome to the Car Wash Management System!"
            )
            self.root.destroy()
            # Iniciar aplicación principal
            main_window = tk.Tk()
            CarWashApp(main_window)
            main_window.mainloop()
        else:
            messagebox.showerror(
                "Access Denied",
                "Invalid username or password.\nCredentials are: programacion / programacion",
            )
            self.txt_pass.delete(0, tk.END)


class CarWashApp:
    """Ventana principal para la gestión del servicio de autolavado."""

    def __init__(self, root: tk.Tk):
        self.root = root
        self.root.title("Car Wash Management System")
        self.root.geometry("680x520")

        # Lista interna de almacenamiento de vehículos (POO)
        self.lista_autos = []

        self.crear_interfaz()

    def crear_interfaz(self):
        # Panel Superior: Registro de Entrada
        frame_input = ttk.LabelFrame(
            self.root, text=" Vehicle Registration (Check-In) ", padding=15
        )
        frame_input.pack(fill="x", padx=15, pady=10)

        ttk.Label(frame_input, text="License Plate:").grid(
            row=0, column=0, padx=5, pady=5, sticky="e"
        )
        self.entry_placa = ttk.Entry(frame_input, width=15)
        self.entry_placa.grid(row=0, column=1, padx=5, pady=5)

        ttk.Label(frame_input, text="Entry Hour (0-24):").grid(
            row=0, column=2, padx=5, pady=5, sticky="e"
        )
        self.entry_ingreso = ttk.Entry(frame_input, width=10)
        self.entry_ingreso.grid(row=0, column=3, padx=5, pady=5)

        ttk.Label(frame_input, text="Hourly Rate ($):").grid(
            row=1, column=0, padx=5, pady=5, sticky="e"
        )
        self.entry_tarifa = ttk.Entry(frame_input, width=15)
        self.entry_tarifa.grid(row=1, column=1, padx=5, pady=5)

        btn_register = ttk.Button(
            frame_input, text="Register Entry", command=self.registrar_vehiculo
        )
        btn_register.grid(row=1, column=2, columnspan=2, padx=5, pady=5, sticky="ew")

        # Panel Central: Lista de Vehículos Activos (Treeview)
        frame_table = ttk.LabelFrame(
            self.root, text=" Active Vehicles in Service ", padding=15
        )
        frame_table.pack(fill="both", expand=True, padx=15, pady=5)

        columns = ("Plate", "Entry Hour", "Hourly Rate")
        self.tree = ttk.Treeview(
            frame_table, columns=columns, show="headings", height=8
        )

        self.tree.heading("Plate", text="License Plate")
        self.tree.heading("Entry Hour", text="Entry Hour (h)")
        self.tree.heading("Hourly Rate", text="Hourly Rate ($)")

        self.tree.column("Plate", anchor="center", width=150)
        self.tree.column("Entry Hour", anchor="center", width=150)
        self.tree.column("Hourly Rate", anchor="center", width=150)

        self.tree.pack(fill="both", expand=True)

        # Panel Inferior: Registro de Salida y Cobro
        frame_checkout = ttk.LabelFrame(
            self.root, text=" Vehicle Departure (Check-Out) ", padding=15
        )
        frame_checkout.pack(fill="x", padx=15, pady=10)

        ttk.Label(frame_checkout, text="Departure Hour (0-24):").pack(
            side="left", padx=5
        )
        self.entry_salida = ttk.Entry(frame_checkout, width=10)
        self.entry_salida.pack(side="left", padx=5)

        btn_checkout = ttk.Button(
            frame_checkout,
            text="Process Departure & Calculate Fee",
            command=self.procesar_salida,
        )
        btn_checkout.pack(side="left", padx=15)

    def registrar_vehiculo(self):
        """Valida y guarda una nueva entrada en la lista interna."""
        placa = self.entry_placa.get().strip()
        str_ingreso = self.entry_ingreso.get().strip()
        str_tarifa = self.entry_tarifa.get().strip()

        if not placa or not str_ingreso or not str_tarifa:
            messagebox.showwarning(
                "Missing Data", "Please fill in all input fields."
            )
            return

        try:
            hora_ingreso = float(str_ingreso)
            tarifa = float(str_tarifa)

            if not (0 <= hora_ingreso <= 24):
                raise ValueError("Entry hour must be between 0 and 24.")
            if tarifa <= 0:
                raise ValueError("Hourly rate must be greater than zero.")

            # Instanciación POO del objeto AutoLavado
            auto = AutoLavado(placa, tarifa)
            auto.registrar_ingreso(hora_ingreso)

            # Guardar en la lista interna
            self.lista_autos.append(auto)

            # Insertar en la tabla visual Tkinter
            self.tree.insert(
                "",
                "end",
                values=(auto.obtener_placa(), f"{hora_ingreso:.2f}", f"${tarifa:.2f}"),
            )

            # Limpiar entradas
            self.entry_placa.delete(0, tk.END)
            self.entry_ingreso.delete(0, tk.END)
            self.entry_tarifa.delete(0, tk.END)

            messagebox.showinfo("Success", f"Vehicle {auto.obtener_placa()} registered successfully!")

        except ValueError as e:
            messagebox.showerror("Invalid Input", f"Error: {e}")

    def procesar_salida(self):
        """Valida la hora de salida, selecciona de la lista e imprime el costo."""
        selected_item = self.tree.selection()

        if not selected_item:
            messagebox.showwarning(
                "Selection Required", "Please select a vehicle from the list to process departure."
            )
            return

        str_salida = self.entry_salida.get().strip()
        if not str_salida:
            messagebox.showwarning(
                "Missing Data", "Please enter the departure hour."
            )
            return

        try:
            hora_salida = float(str_salida)

            # Obtener datos de la fila seleccionada
            item_values = self.tree.item(selected_item, "values")
            placa_seleccionada = item_values[0]

            # Buscar el objeto AutoLavado correspondiente en la lista interna
            auto_obj = None
            for auto in self.lista_autos:
                if auto.obtener_placa() == placa_seleccionada:
                    auto_obj = auto
                    break

            if auto_obj:
                # Validar que la hora de salida sea mayor a la hora de entrada
                if hora_salida <= auto_obj._hora_ingreso:
                    messagebox.showerror(
                        "Time Validation Error",
                        f"Departure hour ({hora_salida}h) must be strictly greater than entry hour ({auto_obj._hora_ingreso}h).",
                    )
                    return

                # Registrar salida y calcular costo
                auto_obj.registrar_salida(hora_salida)
                total_pago = auto_obj.calcular_pago(hora_salida)

                # Mostrar resultado
                messagebox.showinfo(
                    "Service Receipt",
                    f"--- CAR WASH RECEIPT ---\n\n"
                    f"Vehicle Plate: {auto_obj.obtener_placa()}\n"
                    f"Entry Time: {auto_obj._hora_ingreso:.2f} h\n"
                    f"Departure Time: {hora_salida:.2f} h\n"
                    f"Total Hours Charged: {math.ceil(hora_salida - auto_obj._hora_ingreso)} h\n"
                    f"-----------------------------\n"
                    f"TOTAL AMOUNT DUE: ${total_pago:.2f}",
                )

                # Eliminar de la lista interna y de la tabla visual
                self.lista_autos.remove(auto_obj)
                self.tree.delete(selected_item)
                self.entry_salida.delete(0, tk.END)

        except ValueError:
            messagebox.showerror(
                "Invalid Input", "Please enter a valid numeric value for departure hour."
            )


# ==============================================================================
# PUNTO DE ENTRADA PRINCIPAL
# ==============================================================================

if __name__ == "__main__":
    root = tk.Tk()
    app = LoginWindow(root)
    root.mainloop()
