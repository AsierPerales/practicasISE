El plan de pruebas simula el uso de la API por alumnos y administradores, midiendo tiempos y resultados bajo carga. Está organizado en dos grupos de hilos y usa CSV de credenciales, login, extracción de token JWT y peticiones autenticadas.

## Plan general

El nodo raíz es un TestPlan llamado `LAAPIDELAETSIIT` que define variables globales `HOST=192.168.18.173` (es la ip del ordenador host, si queremos ejecutar la API en la maquina virtual y probarla ahí bastaría con cambiar la ip) y `PORT=3000`, usadas como dominio y puerto en las peticiones HTTP. También se configura un elemento de configuración HTTP Defaults para que todas las peticiones hereden host, puerto y el cliente HTTP (`HttpClient4`).

## Autenticación base

Hay un Gestor de Autorización HTTP que define una URL de login `http://${HOST}:${PORT}/api/v1/authentication/login` con usuario `etsiitApi` y contraseña `laApiDeLaETSIITDaLache`.

## Grupo de hilos Alumnos

El Thread Group “Alumnos” lanza 10 hilos, con rampa de 1 segundo y 500 iteraciones, sin reutilizar el mismo usuario entre iteraciones. Carga un CSV `alumnos.csv` con columnas `login,password`, ignorando la primera línea y reciclando las filas para alimentar las variables `${login}` y `${password}`.

Dentro del grupo, el sampler “Login Alumno” hace un POST a `/api/v1/authentication/login` con parámetros de formulario `login` y `password` tomados del CSV. A continuación, un RegexExtractor llamado “Extraer Token” busca en la respuesta el patrón `El token es:(.+)` para extraer el token JWT y guardarlo en la variable `${token}`. Después, un temporizador gaussiano introduce un retardo aleatorio alrededor de 300 ms. Finalmente, el sampler “Recuperar Datos Alumno” hace un POST a `/api/v1/alumnos/alum/` con el parámetro `email=${login}`, y un HeaderManager añade la cabecera `Authorization: Bearer ${token}`.

## Grupo de hilos Administradores

El Thread Group “Administradores” lanza 3 hilos con rampa de 1 segundo y también 500 iteraciones. Carga un CSV `administradores.csv` con `login,password` para credenciales de administrador, e incluye un sampler “Login Admins” que hace POST a `api/v1/authentication/login` con esos datos, seguido de otro RegexExtractor para extraer el token JWT en `${token}`.

Después, se añade otro CSVDataSet que lee `alumnos.csv` como “Log de Alumnos”, asignando la columna `email` a la variable `${email}`. Con esa información, el sampler “Accesos Admins” hace POST a `/api/v1/alumnos/alum/` con `email=${email}`, y el HeaderManager adjunta `Authorization: Bearer ${token}`, simulando accesos de administrador a los datos de distintos alumnos.

## Medición y resultados

El plan añade varios ResultCollector: un “Reporte resumen” (SummaryReport), un “Ver Árbol de Resultados” (ViewResultsTree) y un “Informe Agregado” (Aggregate Report). Se configuran para registrar tiempos, latencias, códigos de respuesta, éxito/fracaso, bytes enviados/recibidos y conteos de hilos, mostrando los resultados en la interfaz mientras corre el test.
