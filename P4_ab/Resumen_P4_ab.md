# Apache Benchmark

*En este ejercicio aprenderemos a evaluar nuestro servidor `apache` usando diferentes versiones del comando `ab` .*

[https://httpd.apache.org/docs/2.4/programs/ab.html](https://httpd.apache.org/docs/2.4/programs/ab.html)

# Probando Apache Benchmark en Debian

---

**OJO! En Debian no tenemos la pila LAMP, por lo que evaluaremos según el manejo de las peticiones a la página de bienvenida de Apache.**

Al principio es abrumador la cantidad de opciones del comando en contraposición con la simpleza de los servidores de Debian y Alma. Por suerte, encontré una guia que me gustó bastante:

[https://nowitsanurag.medium.com/stress-testing-using-apache-bench-ab-98a3f1312246](https://nowitsanurag.medium.com/stress-testing-using-apache-bench-ab-98a3f1312246)

[https://www.tutorialspoint.com/apache_bench/apache_bench_quick_guide.htm](https://www.tutorialspoint.com/apache_bench/apache_bench_quick_guide.htm)

→ En el punto 7 encontré una buena forma de empezar a testear:

```bash
ab -c 100 -n 500 -r 
```

*Donde `-c` mide la concurrencia (solicitudes simultáneas), `-n` el número total de peticiones y `-r` obliga a no abortar en caso de error de socket (por si acaso).*

Esto quiere decir que mandaremos 5 tantas de 100 peticiones simultáneas, básicamente:

```bash
Concurrency Level:      100
Time taken for tests:   0.175 seconds
Complete requests:      500
Failed requests:        0
Total transferred:      5488500 bytes
HTML transferred:       5351500 bytes
Requests per second:    2853.03 [#/sec] (mean)
Time per request:       35.050 [ms] (mean)
Time per request:       0.351 [ms] (mean, across all concurrent requests)
Transfer rate:          30583.75 [Kbytes/sec] received
```

Mejor aún! Podemos usar `-g` para guardar la salida en un archivo para `gnuplot` , o `-v 4` para ver un resultado más detallado (cosa que no haré porque me gustan más las gráficas)

```bash
sudo apt install gnuplot

ab -v 4 -c 100 -n 1000 -r -g ab_Debian_plot.data http://192.168.56.105/

(gnuplot)
	set terminal dumb # Ya que no tenemos gráficos :)
	plot "ab_Debian_plot.data" using 9 w l
```

*El 9 es porque queremos poner la novena columna del archivo `.data` (tiempo de respuesta) en el eje y respecto a el número de peticiones*

# Apache Benchmark en Alma

---

**OJO! en Alma instalamos la pila LAMP, por lo que los resultados no son comparables. Aún así, es interesante ver en que medida la pila es más o menos eficiente comparada con el simple `index.php` por defecto:**

```bash
ab -v 4 -c 100 -n 1000 -r -g ab_Alma_plot.data http://192.168.56.110/
plot "ab_Alma_plot.data" using 9 w l

```

En las imágenes de puede ver una diferencia notable …

# Interpretando Resultados

---

En Alma (LAMP), los tiempos iniciales están entre **30 y 40 ms** y los máximos alrededor de **80 ms**. La curva tiene forma de **escalera ascendente**, reflejando cómo la carga crece con cada solicitud. La **carga del servidor es alta** debido al procesamiento de la pila, y el **cuello de botella** está en **CPU e IO**. En la segunda gráfica se puede ver como se satura el servidor.

En la **segunda imagen** (Apache estático), los tiempos iniciales son muy bajos (**5–10 ms**), aunque los máximos pueden llegar a **150 ms**. La curva es un **horizonte**, mostrando que el servidor mantiene tiempos constantes hasta saturarse. En este caso el **cuello de botella** está en el número máximo de conexiones concurrentes que Apache puede atender antes de poner solicitudes en cola.