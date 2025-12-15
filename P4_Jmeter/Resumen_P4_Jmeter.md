# Jmeter

# La API de la ETSIIT

---

```bash
git clone https://github.com/aguillenATC/ISE-P4App
cd ISE-P4App
docker-compose up
cat docker-compose.yml
```

→ [http://localhost:3000/](http://localhost:3000/) nos muestra esto:

```java
Descripción de la API Restful:
      POST /api/v1/authentication/login
        Parámetros:
          login:<emailUsuario>
          password:<secreto>
        Seguridad:
          Acceso protegido con BasicAuth (etsiitApi:laApiDeLaETSIITDaLache)
        Retorna:
          "El token es:JWTToken"

      POST /api/v1/alumnos/alum/
        Parámetros:
	  email: email para consultar
        Seguridad:
            Token JWT valido en cabecera estandar authorization: Bearer <token>
            Alumnos solo pueden solicitar sus datos. Administradores pueden solicitar cualquier alumno válido
        Retorna:
            Objeto Json con perfil de alumno
```

---

```bash
./pruebaEntorno.sh 
# Nos da un ejemplo de respuesta a una petición de los datos de una alumna 
# en concreto
```

---

**La prueba de JMeter debe:**

- **Simular el acceso concurrente de un número de alumnos no menor a 10 y un número de administradores no menor de 3.**
- **Parametrizar el "EndPoint" del servicio mediante variables para la
dirección y puerto del servidor. Emplee "User Defined Variables" del
Test Plan.**
- **Definir globalmente las propiedades de los accesos Http y la
Autenticacion Basic. Emplee HTTP Request Defatuls y HTTP Authorization
Manager.**
- **Los accesos de alumnos y administradores se modelarán con 2 Thread
Groups independientes. La carga de accesos de administradores se
modelará empleando el registro de accesos del archivo apiAlumno.log**

# Instalando y usando `Jmeter`

---

[Fuente 1](https://jmeter.apache.org/usermanual/build-web-test-plan.html)

```bash
sudo apt update
sudo apt install default-jre
java -version

wget https://dlcdn.apache.org/jmeter/binaries/apache-jmeter-5.6.3.tgz
tar -xvzf apache-jmeter-5.6.3.tgz

# Esto era por salud mental, no es necesario
sudo mv apache-jmeter-5.6.3 /opt/jmeter
sudo ln -s /opt/jmeter/bin/jmeter /usr/local/bin/jmeter

jmeter
```

[Fuente 2](https://jmeter.apache.org/usermanual/get-started.html)

[Fuente 3](https://www.youtube.com/watch?v=817zU_bXh9Y&list=PLUDwpEzHYYLs33uFHeIJo-6eU92IoiMZ7)

# Creando el `TestPlan` para la API

---

*Una vez tenemos el entorno listo, toca crear el test de carga que nos piden en el enunciado. Esto implica:*

- Definir valores por defecto de las peticiones HTTP, así como la autorización con BasicAuth
- Definir el grupo de hebras de **Alumnos:**
    - Credenciales
    - Login
    - Extraer Token
    - Petición con el token de los datos **del mismo alumno**
- Definir el grupo de hebras de **Administradores:**
    - Credenciales, Login, Token …
    - Credenciales de los alumnos
    - Petición de datos de **cualquier alumno**

### A tener en cuenta

---

*Debido a que las versiones del guión y de la API son inconsistentes (las peticiones del log no son correctas y el path para acceder a los datos de los alumnos no es correcto) me vi obligado a utilizar csv de nuevo en vez de utilizar el log sampler, por la inconsistencia ya mencionada.* 

Por lo que en **mi** versión se utiliza el csv de alumnos (`alumnos.csv`) para extraer los distintos `emails` que se añadiran en el **cuerpo** de la petición **POST (no GET como en el log, donde además el `email` se incluye en la url)  a la ruta `api/v1/alumnos/alum` (en vez de** `alumnos/alumno, como se lee en el guión` ).

Por ende, la extracción de las credenciales, el inicio de sesión, la extracción del token y la petición de los datos de los alumnos se hace con el mismo procedimiento tanto en **alumnos como en administradores, exceptuando el reuso de el archivo `alumnos.csv` para simular el log de consulta de los administradores.**

*[Explicación del TestPlan](./test_plan)*

*[Capturas del TestPlan y los cambios hechos](./capturas)*

# Ejecución y resultado del Test

---

1. Instalar docker en la MV 
2. Desplegar la API
3. **OJO ! HAY QUE CAMBIAR LA IP DEL HOST Y LA RUTA DE LOS ARCHIVOS REQUERIDOS (alumnos.csv y administradores.csv)**
4. Ejecutamos el plan en jmeter desde el ordenador host a la MV
5. Comprobamos ***Reporte Resumen y Arbol de Resultados***

*[Resultados en el host](./resultados)*
