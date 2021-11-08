from collections import namedtuple
from typing import List
import re
import os
import csv
import pandas as pd
import datetime
import time
import sqlite3
from sqlite3 import Error
#Agrega un registro en una tabla

import sys

repetidor = True
objecto_registrado = {}
Registro = namedtuple("Registro", ("descripcion","cantidad","precio","fecha"))
registado_completo = {}
separador = ("-" * 20) + "\n"
total_monto = 0

while repetidor:
    print("MENU REGISTRO DE PERSONA")
    print("1: Agregar un registro ")
    print("2: Registro especifico")
    print("3: exportar a csv")
    print("4: crear tabla")
    print("5: salir")
    print(separador)
    resp = input("Ingrese una opción: ")

    if resp == "1":
    #recorrido del diccionario 
        Registros_Marcados = []
        otro_registro = "S"
        folio = input("Marque su folio por favor: ")
        #mencionar que si ya hay un folio registrado no perimita registrar
        if not folio in objecto_registrado.keys():
            fecha = input("Dime una fecha: ")
            procesado = datetime.datetime.strptime(fecha, "%d/%m/%Y").date()
            while otro_registro.upper()=="S":
                descripcion = input("Marque el producto que desea llevarse: ")
                cantidad = int(input("La cantidad del producto que desea llevar: "))
                precio = int(input("El precio de sus productos: "))
            
                Registro_vendido = Registro(descripcion,cantidad,precio,fecha)
                Registros_Marcados.append(Registro_vendido)
            
                total = (cantidad * precio )
                total_monto = total_monto + total
                iva = (total_monto * 0.16)
                total_iva = total_monto + iva
            
                print(f"el monto que va a pagar es: {total_iva}")
                print(f"y su iva sera: {iva}")
                print("¿Desea Hacer otro registro?  [S/N]")
                otro_registro=input()
                ##Las Listas de refacciones folio, fecha, descripcion, cantidad, precio, total
                registado_completo[fecha] = [Registros_Marcados, descripcion,cantidad, precio, total, iva, total_iva, fecha, folio]
                ##Las Listas de refacciones folio, fecha, descripcion, cantidad, precio, total
                objecto_registrado[folio] = [Registros_Marcados, descripcion,cantidad, precio, total, iva, total_iva, fecha, folio]
                # Insertar registros a la tabla
                registro = folio,cantidad,precio,total_monto,total_iva
                try:
                    with sqlite3.connect("Registros.db") as conn:
                        c = conn.cursor()
                        c.execute("INSERT INTO Registros VALUES(?,?,?,?,?)",registro)
                except Error as e:
                    print(e)
                except Exception:
                    print(f"Se produjo el siguiente error: {sys.exc_info()[0]}")
                finally:
                    if conn:
                        conn.close()
        else:
            print("folio ya esta registrado")
            
      
    elif resp == "2":
        #Si en el diccionario no esta la clave te dara la opcion de fecha no registrado
        buscar = input("Dime una fecha: ")
        if buscar in registado_completo.keys():
            print(f"La fecha es: {registado_completo[buscar][7]} ")
            print(f"La folio es: {registado_completo[buscar][8]}")
            for buscar_producto in registado_completo[buscar][0]:
                print(f"La descripcion del producto es: {registado_completo[buscar][1]}")
                print(f"la cantidad que se llevara es: {registado_completo[buscar][2]}")
                print(f"El precio del producto es: {registado_completo[buscar][3]}")
            print(f"El monto del producto es: {registado_completo[buscar][6]} ")
            print(f"El iva sera: {registado_completo[buscar][5]} ") 
            print(separador)
        else:
            print("No se encontro fecha con la cual se registro")
            
    elif resp == "3":
        #importar a csv de los registros registrados
        List = [descripcion, cantidad, fecha, precio, total, total_iva]
        print (str(List[0]) + "\t" + str(List[1]) + "\t" + str(List[2]))
        with open("Registros_subidos.csv", "w", newline="") as cosas_registradas:
            completos = csv.writer(cosas_registradas)
            completos.writerow(("descripcion","cantidad","precio", "total", "total_iva"))
            completos.writerows([(descripcion, cantidad, precio, total, total_iva) for fecha in registado_completo.items()])
            print(f"---Grabado exitoso en-----> {os.getcwd()}")
            
    elif resp == "4":
        #importar a sql lite
        try:
            with sqlite3.connect("Registros.db") as conn:
                c = conn.cursor()
                c.execute("CREATE TABLE IF NOT EXISTS Registros (FOLIO INTEGER PRIMARY KEY, CANTIDAD TEXT NOT NULL, PRECIO REAL NOT NULL, TOTAL REAL NOT NULL, TOTAL_IVA REAL NOT NULL)")
                print("Tabla creada exitosamente")
        except Error as e:
            print(e)
        except Exception:
            print(f"Se produjo el siguiente error: {sys.exc_info()[0]}")
        finally:
            if conn:
                conn.close()
        
    elif resp == "5":
        repetidor = False
