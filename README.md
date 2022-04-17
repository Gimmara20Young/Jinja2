# Jinja2
#API de tipo REST

#En base a la aplicación desarrollada en Desplegar Flask sobre Gunicorn y Nginx, debe sustituir la arquitectura de conexión a una base de datos no relacional a manejarla por un API de tipo REST
#En base a la aplicación desarrollada en la Sustitución de CLI por Web, debe asegurarse de utilizar el motor de plantilla Jinja2
crearemos 3 carpetas para ejecucion del mismo, main.py; diccionario(ya la teniamos),base_datos.py

#en la que llamamos main, el siguiente codigo
from flask import Flask, render_template, request
import sqlite3 as sql
app = Flask(__name__)


@app.route('/')
def nueva_palabra():
   return render_template('diccionario.html')

@app.route('/significadoec',methods = ['POST', 'GET'])
def significadoec():
   if request.method == 'POST':
      try:
         palabra = request.form['palabra']
         significado = request.form['significado']
         
         with sql.connect("diccionario_slang.db") as con:
            cur = con.cursor()
            
            cur.execute("INSERT INTO diccionario (palabra,significado) VALUES (?,?)",(palabra,significado) )
            
            con.commit()
            msg = "Palabra agregada"
      except:
         con.rollback()
         msg = "Error"
      
      finally:
         return render_template("resultado.html", msg = msg)
         con.close()

# Para observar las palabra ingresadas
@app.route('/lista')
def lista():
   con = sql.connect("diccionario_slang.db")
   con.row_factory = sql.Row
   
   cur = con.cursor()
   cur.execute("select * from diccionario")
   
   rows = cur.fetchall();
   return render_template("lista.html",rows = rows)

if __name__ == '__main__':
   app.run(debug = True)
   
   #diccionario
   import redis

# Abrir el servidor de redis primero 
client = redis.Redis(host = '127.0.0.1', port = 6379, db=0)

def agregar_palabra(palabra, significado):
	client.set(palabra,significado)

def editar_palabra(palabra):
    x=client.exists(palabra)
    if x==1:
        nuevo_significado= input("Ingrese el nuevo significado: ")
        client.set(palabra,nuevo_significado)
    else:
        print("Esta palabra no existe")

def buscar_significado_palabra(palabra):
	print(client.get(palabra))

def obtener_palabras():
    print(client.keys('*'))

def eliminar_palabra(palabra):
    client.delete(palabra)

def principal():

    menu = """
1) Agregar nueva palabra
2) Editar palabra existente
3) Buscar significado de palabra
4) Ver listado de palabras
5) Eliminar palabra existente
6) Salir
Elige: """
    eleccion = ""
    while eleccion != "6":
        eleccion = input(menu)
        if eleccion == "1":
            palabra = input("Ingresa la palabra: ")
            significado = input("Ingresa el significado: ")
            agregar_palabra(palabra,significado)
            print("Palabra agregada")

        if eleccion == "2":
            palabra = input("Ingresa la palabra que quiere editar: ")
            editar_palabra(palabra)

        if eleccion == "3":
            palabra = input("Ingresa la palabra del significado que quieres ver: ")
            buscar_significado_palabra(palabra)

        if eleccion == "4":
            obtener_palabras()

        if eleccion == "5":
            palabra = input("Ingrese la palabra a eliminar: ")
            eliminar_palabra(palabra)
            print("Palabra eliminada")


if __name__ == '__main__':
    principal()

#base_datos.py
import sqlite3

conn = sqlite3.connect('diccionario_slang.db')
conn.execute('CREATE TABLE diccionario (palabra TEXT, significado TEXT)')
conn.close()

#En base a la aplicación desarrollada en Desplegar Flask sobre Gunicorn y Nginx, debe sustituir la arquitectura de conexión a una base de datos no relacional a manejarla por un API de tipo REST
archivo main

from flask import Flask, request, jsonify, Response
from flask_pymongo import PyMongo
from bson import json_util

app = Flask(__name__)
app.config['MONGO_URI']="mongodb://localhost/sustituir"
mongo=PyMongo(app)

@app.route('/', methods=['POST'])

def agregar_palabra():

	palabra = request.json['palabra']
	significado = request.json['significado']

	if palabra and significado:
		id=mongo.db.slangdic.insert(
				{'palabra':palabra, 'significado':significado}
			)
		response = {
			'id':str(id),
			'palabra':palabra,
			'significado':significado,
		}
		return response
	else:
		return not_found()

	return {'message':'received'}

@app.route("/",methods=['GET'])
def get_palabras():
	palabras = mongo.db.slangdic.find()
	response=json_util.dumps(palabras)
	return Response(response, mimetype='application/json')




@app.errorhandler(404)
def not_found(error=None):
	response = jsonify({
		'message': 'Resource Not Found: ' + request.url,
		'status': 404
	})
	return response





if __name__ == "__main__":
	app.run(port=5022,debug=True
