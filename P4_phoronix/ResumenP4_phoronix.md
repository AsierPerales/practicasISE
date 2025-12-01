# Benchmarking Con Phoronix

# Benchmarking con Phoronix

---

*El objetivo es aplicar a nuestras máquinas (tanto Debian como Alma) una serie de `benchmarks` para medir su rendimiento en distintos ámbitos.*

*Para ello, utilizaremos la suite `phoronix` y seleccionaremos varios de los benchmarks que nos proporcione esta, así como otros que sea preciso instalar.*

→ Nota : por comodidad, todo el proceso siguiente se relata en el contexto de Debian. Después comentaré las diferencias de Alma respecto a este.

### Instalación y prueba de Phoronix Test Suite

---

- [https://github.com/phoronix-test-suite/phoronix-test-suite/blob/master/documentation/phoronix-test-suite.md](https://github.com/phoronix-test-suite/phoronix-test-suite/blob/master/documentation/phoronix-test-suite.md)

```bash
git clone https://github.com/phoronix-test-suite/phoronix-test-suite.git
cd phoronix-test-suite 
sudo ./install-sh
```

Dada la documentación, parece ser que basta con este simple comando para instalar, ejecutar y analizar un benchmark:

```bash
phoronix-test-suite benchmark smallpt
## También trastee con esta opción, sin resultados satisfactorios:
run-random-tests
```

- Resultado : [https://openbenchmarking.org/result/2512013-NE-PRUEBA11621](https://openbenchmarking.org/result/2512013-NE-PRUEBA11621)

### Utilizando PTS para más cosas

---

Phoronix no solamente permite descargar y ejecutar benchmarks públicos, sino además proporciona opciones de lotes de benchmarks con:

```bash
phoronix-test-suite batch-setup
## Elegi las opciones por defecto y las recomendadas por batch-run sin argumentos
phoronix-test-suite batch-run llama-cpp smallpt (previamente instalados!)
```

El `batch` descarga tests, los ejecuta  y guarda resultados automáticamente !

- Resultado : [https://openbenchmarking.org/result/2512015-NE-PRUEBA2BA38](https://openbenchmarking.org/result/2512015-NE-PRUEBA2BA38)

También probé la opción de hacer que los resultados se exporten en diferentes formatos:

```bash
result-file-to-html prueba1
result-file-to-pdf prueba1
...
```

### Diferencias respecto a Alma

---

Todo el proceso es análogo a Alma, exceptuando que la instalación es más sencilla con:

```bash
sudo dnf install phoronix-test-suite
phoronix-test-suite benchmark smallpt
```

*Ojo : requiere instalar el comando `tar` antes*

- Resultado : [https://openbenchmarking.org/result/2512014-NE-PRUEBA16267](https://openbenchmarking.org/result/2512014-NE-PRUEBA16267)