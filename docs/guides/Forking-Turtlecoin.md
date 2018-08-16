# Bifurcando TurtleCoin

Entonces quieres bifurcar TurtleCoin, ¿eh?

Esta guía lo ayudará a cambiar las secciones necesarias del código para configurar su moneda a su gusto.

En primer lugar, vamos a hablar rápidamente sobre *licencias*.

## Licencias

Cuando estés hurgando en la base de código, es posible que veas algo como esto:

```
// Copyright (c) 2012-2017, The CryptoNote developers, The Bytecoin developers
// Copyright (c) 2014-2018, The Monero Project
// Copyright (c) 2018, The TurtleCoin Developers
//
// Please see the included LICENSE file for more information.
```

En la parte superior de cada archivo. Le preguntamos cuándo nos bifurca para asegurarnos de que deja intactos estos encabezados de licencia. Cuando estamos programando en bases de código como este, estamos sobre los hombros de gigantes, y es a la vez cortés, y un requisito legal de las licencias utilizadas, acreditar a los otros contribuidores al código.

Como sabrá, el documento técnico de CryptoNote establece la especificación inicial de las monedas de CryptoNote. Los desarrolladores de ByteCoin crearon la primera implementación de esta especificación trabajando en estrecha colaboración con los desarrolladores de CryptoNote, y esto es lo que se ha convertido hoy en ByteCoin. Esto se bifurcó desde el principio para convertirse en Bitmonero, y luego en Monero. Los desarrolladores de Monero han contribuido con trozos de código significativos a la base de código original de ByteCoin, y por supuesto, mantienen sus propios repositorios y códigos.

El proyecto ForkNote se creó como una bifurcación de ByteCoin, para crear una forma de bifurcar ByteCoin, separando fácilmente las constantes y cadenas necesarias en un archivo separado.

TurtleCoin luego bifurcó el Proyecto Forknote e hizo nuestros propios cambios, algunos de los cuales fueron luego devueltos a ForkNote.

Por lo tanto, asegúrese de buscar y reemplazar para que todas estas líneas permanezcan intactas. Por supuesto, puede comenzar a agregar su propia línea de derechos de autor, por ejemplo,

```
// Copyright (c) 2012-2017, The CryptoNote developers, The Bytecoin developers
// Copyright (c) 2014-2018, The Monero Project
// Copyright (c) 2018, The TurtleCoin Developers
// Copyright (c) 2018, My Super Cool Coin Developers
//
// Please see the included LICENSE file for more information.
```

Ok, ahora entendemos las licencias, ¡obtengamos el código!


## El proceso actual de bifurcación

* La forma más fácil de bifurcar TurtleCoin es comenzar creando una cuenta de GitHub. Puede hacer esto [aquí](https://github.com/join) si todavía no tiene una cuenta.

* Asegúrate de haber iniciado sesión. Luego, dirígete al repositorio de TurtleCoin y presiona `Fork` en la esquina superior derecha.

* Debería verse algo como esto:

  ![Fork it!](https://i.imgur.com/ZNUS8ED.png)

* Luego, si desea obtener el código fuente en su computadora. 
  Dirígete al repositorio que acabas de abrir, y haz clic en el botón `Clone or Download`.
  Queremos clonar el código fuente, para que podamos hacer cambios y luego volver a subirlo a GitHub.

* Debería verse algo como esto:

  ![Clone it!](https://i.imgur.com/UlqtvF6.png)

* Ahora, tome esta URL que se muestra y cópiela.

* A continuación, tendrá que descargar un cliente de Git. 
  Si eres un usuario de Windows, te sugiero [Git para Windows](https://gitforwindows.org/),
  mientras que si eres un usuario de Linux o Mac, debes tenerlo ya instalado,
  o instalarlo desde tus repositorios o con brew.

* Una vez que hayas abierto git bash o una terminal, deberías poder escribir `git clone URL`, donde URL es el enlace que         copiaste anteriormente, desde GitHub.

* Debería verse algo como esto:

![Bop it!](https://i.imgur.com/Fv435iR.png)

* Genial, ahora el código está en nuestra computadora. ¡Estamos casi listos para comenzar a cambiarlo!


A continuación, me gustaría hablar rápidamente sobre las *unidades atómicas*. Puede saltear esta sección si ya es un profesional.

## Unidades Atómicas

Todas las constantes en CryptoNoteConfig.h y la mayor parte del código, que se refiere a la cantidad actual de monedas, se encuentra en *Unidades Atómicas*.

**¿Qué significa esto?**

Bueno, si estamos hablando de TurtleCoin, tenemos 2 decimales. 
Eso significa que para obtener las unidades atómicas de una cantidad de TRTL, necesitamos multiplicar la cantidad por 100 o 10 ^ 2.

Si tenemos 10.23 TRTL, esto es 1023 en unidades atómicas. Las unidades atómicas no tienen punto decimal y, por lo tanto, se pueden representar como un número entero en el código.
Algunas monedas diferentes tienen un nombre especial para sus unidades atómicas, para que sea más fácil hablar de ellas. En Bitcoin, esto se llama *satoshi*, y en TurtleCoin, se llama *shell*.

**¿Por qué es esto útil?**

Las computadoras no pueden almacenar con precisión los números de coma flotante (o números con un punto decimal), por lo que realizar cálculos matemáticos suele ser problemático.

Por ejemplo, supongamos que queremos dividir 10.00 TRTL entre 3 personas. Si actúa 10.00 / 3en un lenguaje de programación, probablemente verá un resultado similar a 3.3333333333333335. ¿De dónde salió ese 5? La computadora es incapaz de almacenar este número en su cantidad total de decimales.

**¿Cómo arreglamos esto?**

Ahora, demostré cómo podemos resolver este problema, con *unidades atómicas*.
De nuevo, tenemos 10.00 TRTL, y queremos dividirlo entre 3 personas.
Recuerde que para obtener el número de unidades atómicas, necesitamos multiplicar por 100. 
Entonces, tenemos un valor de 1000 en unidades atómicas. Ahora podemos realizar una *división entera* en su lugar.
Vamos a realizar 1000 / 3- asegúrate de que estés usando la *división integral* en tu lenguaje de programación y deberías obtener un resultado de 333.

Eso es correcto, cada persona solo puede obtener 333 unidades, o, si queremos convertir de nuevo a TRTL, 3.33 TRTL.
Es posible que haya notado que nos falta una unidad atómica, o 0.01 TRTL.
Tenemos que usar los residuos cuando se trata de valores atómicos, ya que una unidad atómica ya no puede dividirse.

Entonces, el código puede parecerse a esto, cuando se usan unidades atómicas.

```cpp
/* 10.00 TRTL in atomic units */
float trtl = 10.00;
int shells = static_cast<int>(trtl) * 100;

int numPeople = 3;

int shellsPerPerson = shells / numPeople;
int remainder = shells % numPeople;

std::cout << "Splitting " << trtl << " TRTL" between << numPeople << " gives "
          << (shellsPerPerson / 100) << " TRTL each, with " << (remainder / 100)
          << " TRTL spare." << std::endl;
```

Usualmente, *siempre* usaremos unidades atómicas en nuestro código, y solo volveremos a la representación esperada con un punto decimal cuando imprimamos cosas al usuario.

Bien, sabemos qué son las unidades atómicas, ¡vamos a cambiar algunas cosas!

## Cambiar el nombre de tus binarios, es decir, CMakeLists.txt

Comenzaremos con algunas cosas aburridas, cambiando el nombre de tus binarios.

Abra CMakeLists.txt en la carpeta `src/`.

Tenga en cuenta que también hay un CMakeLists.txt en el directorio raíz, ¡este no es el archivo que desea!

Las líneas que queremos cambiar están en la parte inferior del archivo.

```
set_property(TARGET Daemon PROPERTY OUTPUT_NAME "TurtleCoind")
set_property(TARGET zedwallet PROPERTY OUTPUT_NAME "zedwallet")
set_property(TARGET PaymentGateService PROPERTY OUTPUT_NAME "walletd")
set_property(TARGET PoolWallet PROPERTY OUTPUT_NAME "poolwallet")
set_property(TARGET Miner PROPERTY OUTPUT_NAME "miner")
```

Para cambiar el nombre de un binario, cambie la cadena final en una de estas líneas. Por ejemplo:

`set_property(TARGET Daemon PROPERTY OUTPUT_NAME "AppleCoind")`

Guarde el archivo y vuelva a compilar.


## Cambiar los números de versión

Otro aburrido: cambiar los números de versión. Esto es lo que 
aparece cuando inicias TurtleCoind, por ejemplo `Welcome to TurtleCoin v0.6.4.1264`

Abre `src/version.h.in`.

Los valores son bastante auto explicativos. Establezca `PROJECT_NAME` el nombre de su moneda,
`PROJECT_SITE` su sitio web de monedas y `PROJECT_COPYRIGHT` su cadena de derechos de autor.

A continuación, los números de versión.

`APP_VER_MAJOR` determina la primera parte de la versión. 
Por lo general, incrementa este valor si realiza un cambio sustancial en su código,
lo que potencialmente puede dañar las API, los formatos de billetera, los formatos de cadena de bloques, etc.

`APP_VER_MINOR` determina el segundo número de la versión. determina el segundo número de la versión. Esto generalmente se incrementa para actualizaciones significativas, como las horquillas duras.Esto generalmente se incrementa para actualizaciones significativas, como las bifurcaciones duras.

`APP_VER_BUILD` determina el número final de la versión. 
Esto generalmente se incrementa por pequeños cambios, como correcciones de errores, aceleraciones, pequeños cambios en la billetera, etc.

Estos números se combinan para formar el número de versión completo, en el formato:

`APP_VER_MAJOR`.`APP_VER_MINOR`.`APP_VER_BUILD`.

Por ejemplo, si establecemos los 3 valores en 0, 6 y 4, terminaríamos con un número de versión de 0.6.4.

Un buen comienzo para su proyecto es 0.0.1, entonces

```
#define APP_VER_MAJOR 0
#define APP_VER_MINOR 0
#define APP_VER_REV 1
```

## CryptoNoteConfig.h

Ok, ahora en las cosas divertidas!

Abra el archivo `CryptoNoteConfig.h`, ubicado en la carpeta `src`.

Comencemos en la parte superior. Solo nos enfocaremos en las constantes que deben cambiar, ya que algunas de ellas están bien para mantenerse como están.


#### `const uint64_t DIFFICULTY_TARGET = 30; // seconds`<br>

Así de rápido quieres que sean los bloques. En TurtleCoin, tenemos bloques en promedio cada 30 segundos. 
Si desea que los bloques sean cada 2 minutos, debe configurar esto para que sea:

- `const uint64_t DIFFICULTY_TARGET = 120; // seconds`  
<br>

#### `const uint64_t CRYPTONOTE_PUBLIC_ADDRESS_BASE58_PREFIX = 3914525;`<br/>

Esto define con qué comenzarán las direcciones. En TurtleCoin, esto decodifica a `TRTL`.

Entonces, ¿cómo obtenemos este prefijo? Podemos usar esta [herramienta práctica](https://cryptonotestarter.org/tools.html).

![Prefix Generator](https://i.imgur.com/F2ZARZ9.png)

Desplácese hasta el generador de prefijos e ingrese el prefijo de su dirección deseada.
Tenga en cuenta que algunos caracteres están desactivados, por lo que es posible que no pueda obtener el prefijo perfecto.
Por ejemplo, `aPPLE` está permitido, pero `APPLE` no lo esta.

Puede observar que el prefijo de TRTL es un número, mientras que la salida de este generador no lo es.
Estos valores están en formato hexadecimal y se decodificarán automáticamente en un número cuando se compilen,
aunque puede usar cualquier conversor hexadecimal a decimal estándar para cambiar el prefijo a un número si lo prefiere.

Entonces, si quisiéramos que nuestro prefijo fuera 'aPPLE', lo configuraríamos así:

- `const uint64_t CRYPTONOTE_PUBLIC_ADDRESS_BASE58_PREFIX = 0x1e1f8cc7;`
<br>


#### `const uint32_t CRYPTONOTE_MINED_MONEY_UNLOCK_WINDOW = 40;`<br/>

Este valor define cuántos bloques se deben seguir en la cadena actual
antes de liberar la recompensa por extraer un bloque para gastar.

Sugerimos que establezca este valor para que sea aproximadamente igual a 20 minutos - en
el caso de TurtleCoin eso es exactamente lo que tenemos - 40 bloques * 30 segundos = 20
minutos.

Si tiene un tiempo de bloqueo de 2 minutos, por ejemplo, estableceríamos este valor
en 10.

- `const uint32_t CRYPTONOTE_MINED_MONEY_UNLOCK_WINDOW = 10;`<br/>


#### `const uint64_t MONEY_SUPPLY = UINT64_C(100000000000000);`<br/>

Esta línea es bastante significativa. Determina el suministro máximo de monedas que tendrá su criptomoneda.
En TurtleCoin, esto es 1 billón de TRTL, pero como se mencionó anteriormente, todos estos valores están en *unidades atómicas*,
por lo que este valor también incluye la cantidad después del punto decimal.
Por lo tanto, este valor es 1 billón * 100, ya que TurtleCoin tiene 2 lugares después del punto decimal.

Por lo tanto, le sugerimos que decida cuántos lugares decimales tiene su moneda, antes de completar este valor.

Si usted quiere que su moneda posea 6 decimales, y la emisión total de diez mil (10.000),
el valor MONEY_SUPPLY sería: `10^6 (or 1,000,000) * ten thousand (10,000) = 10000000000`.

Entonces, mostraríamos este valor para darnos:

- `const uint64_t MONEY_SUPPLY = UINT64_C(10000000000);`<br/>


#### `const uint32_t ZAWY_DIFFICULTY_BLOCK_INDEX = 187000;`<br/>

A continuación tenemos la altura en la que queremos que el primer algoritmo de dificultad zawy se active.
El algoritmo de dificultad determina qué tan difícil es explotar un bloque en función de la velocidad con la que entraron los bloques anteriores.

Este algoritmo de dificultad ha encontrado algunos defectos con él, y ahora se usa un algoritmo zawy más nuevo.
Sugeriría que establezca este valor en `0`, para activarlo instantáneamente,
y luego cambiaremos al algoritmo zawy más nuevo directamente a la altura de `1`.

- `const uint32_t ZAWY_DIFFICULTY_BLOCK_INDEX = 0;`<br/>


#### `const uint64_t LWMA_2_DIFFICULTY_BLOCK_INDEX = 620000;`<br/>

Este valor establece la altura del algoritmo zawy más nuevo que se mencionó anteriormente.
Le sugiero que establezca este valor en `1`, para activarlo instantáneamente después del algoritmo anterior.
Este algoritmo de dificultad es mucho más resistente a la minería de pulsos y al time warping.

- `const uint64_t LWMA_2_DIFFICULTY_BLOCK_INDEX = 1;`<br/>


#### `const unsigned EMISSION_SPEED_FACTOR = 25;`<br/>

Este valor define qué tan rápido se emitirá el suministro máximo.
Un valor más pequeño significa que el suministro se emitirá más rápido,
y un valor más alto significa que tomará más tiempo para que se libere el suministro.
Puede volver a utilizar el sitio web [vinculado anteriormente](https://cryptonotestarter.org/tools.html)
para experimentar con diferentes valores de emisión y cómo afectarán el tiempo que tarda en distribuirse 
el suministro.

Si quisiéramos una emisión rápida, podríamos establecer esto en un valor como 21.


- `const unsigned EMISSION_SPEED_FACTOR = 21;`<br/>


#### `const size_t CRYPTONOTE_DISPLAY_DECIMAL_POINT = 2;`<br/>

Este valor define cuántos números hay después del punto decimal en su moneda.
En TurtleCoin, este valor es 2, por lo que tenemos cantidades como 10.23 TRTL.
Si establecemos esto en 6, tendríamos una cantidad como 10.234567 TRTL en su lugar. 
Recuerde, como se mencionó anteriormente, esto afecta su oferta de dinero y otros parámetros que dependen de las unidades atómicas.

- `const size_t CRYPTONOTE_DISPLAY_DECIMAL_POINT = 6;`<br/>


#### `const uint64_t MINIMUM_FEE = UINT64_C(10);`<br/>

Este valor define cuál es la tarifa mínima que un usuario debe gastar para enviar una transacción.
Tenga en cuenta que esto no se aplica a las transacciones de fusión.

Un valor más bajo permite transacciones más baratas, pero puede significar que su red puede recibir 
correo no deseado con muchas pequeñas transacciones.

Por lo general, se desea un punto medio, aunque opcionalmente podría plantearse en una fecha posterior.

Este valor se define nuevamente en *unidades atómicas*,
por lo tanto, multiplique su tarifa mínima deseada por 10 * la cantidad de números después del punto decimal en su moneda.


- `const uint64_t MINIMUM_FEE = UINT64_C(1000);`<br/>


#### `const uint64_t MINIMUM_MIXIN_V1 = 0;`<br/>

Esta sección cubrirá todos estos, porque todos están relacionados:

```
const uint64_t MINIMUM_MIXIN_V1                              = 0;
const uint64_t MAXIMUM_MIXIN_V1                              = 100;
const uint64_t MINIMUM_MIXIN_V2                              = 7;
const uint64_t MAXIMUM_MIXIN_V2                              = 7;

const uint32_t MIXIN_LIMITS_V1_HEIGHT                        = 440000;
const uint32_t MIXIN_LIMITS_V2_HEIGHT                        = 620000;

const uint64_t DEFAULT_MIXIN = MINIMUM_MIXIN_V2;
```

Estos establecen los valores mixin que están permitidos en la red, y en qué altura se aplican.
Si quieres una moneda privada desde el principio,
Te sugiero que establezcas MIXIN_LIMITS_V1_HEIGHT en 0 y MIXIN_LIMITS_V2_HEIGHT en 1.
Luego, establece los valores MINIMUM_MIXIN_V1 / V2 y MAXIMUM_MIXIN_V1 / V2 según corresponda.

No establezca un valor que sea *demasiado* alto, de lo contrario, su red tendrá dificultades para procesar las transacciones.
No debe establecer el mínimo más de 100, y los valores en su sano estarán en algún lugar en el rango de 0 a 30.

`DEFAULT_MIXIN` se usa para establecer el valor de mixin usado en zedwallet, por lo tanto ajústelo a un valor razonable.

¡Asegúrate de que tu mínimo sea menor o igual a tu máximo!

```
const uint64_t MINIMUM_MIXIN_V1                              = 0;
const uint64_t MAXIMUM_MIXIN_V1                              = 100;
const uint64_t MINIMUM_MIXIN_V2                              = 7;
const uint64_t MAXIMUM_MIXIN_V2                              = 7;

const uint32_t MIXIN_LIMITS_V1_HEIGHT                        = 0;
const uint32_t MIXIN_LIMITS_V2_HEIGHT                        = 1;

const uint64_t DEFAULT_MIXIN = MINIMUM_MIXIN_V2;
```
<br/>


#### `const uint64_t DEFAULT_DUST_THRESHOLD  = UINT64_C(10);`<br/>


Esta sección cubrirá todos estos, porque todos están relacionados:

```
const uint64_t DEFAULT_DUST_THRESHOLD = UINT64_C(10);
const uint64_t DEFAULT_DUST_THRESHOLD_V2 = UINT64_C(0);

const uint32_t DUST_THRESHOLD_V2_HEIGHT = MIXIN_LIMITS_V2_HEIGHT;
```

El umbral de polvo determina la cantidad más pequeña que se puede enviar en una transacción,
y puede generar una gran cantidad de pequeñas cantidades que se acumulan en su billetera y que no pueden enviarse.

Establecer el umbral de polvo a cero evita esto, pero tiene un efecto secundario de hacer las transacciones un poco más grandes.

Sugiero configurar DUST_THRESHOLD_V2_HEIGHT en 0, para hacer pequeñas cantidades siempre gastables.

- `const uint32_t DUST_THRESHOLD_V2_HEIGHT = 0;`<br/>


#### `const uint32_t UPGRADE_HEIGHT_V4 = 350000;`<br/>


Este valor determina cuándo el algoritmo de minería pasará a Original CryptoNight,
también conocido como CN v0, a CryptoNight Lite v1, también conocido como CryptoNight Lite v7.

Original CN actualmente permite los ASIC en la red, por lo que le sugerimos que configure esta altura de actualización en 3,
para cambiar casi instantáneamente al resistente de ASIC, CryptoNight Lite v1.

Sin embargo, si prefiere permanecer en CryptoNight original,
puede encontrar dónde se usa esta variable y aplicar un parche, o bien,
puede configurar la altura de actualización como `std::numeric_limits<uint32_t>::max()`

(Es probable que necesites agregar `#include <limits>` a la parte superior del archivo si eliges esta opción)

- `const uint32_t UPGRADE_HEIGHT_V4 = 3; // Upgrade height for CN-Lite Variant 1 switch.`<br/>




#### `const uint64_t FORK_HEIGHTS[] =`<br/>

Esta variable es utilizada por el comando `status` en zedwallet y TurtleCoind para notificar a los usuarios cuando 
se aproxima una bifurcación, o su software está desactualizado.
Le sugerimos que configure algunas bifurcaciones con regularidad,
si necesita actualizar el software para que los usuarios puedan saber cuándo pueden esperarlo.

Si no desea pre-preparar ninguna bifurcación, puede vaciar la lista.

Esto configurará el comando de estado para notificar una bifurcación a 100k y 300k bloques.

- ```
  const uint64_t FORK_HEIGHTS[] =
  {
    100000,
    300000
  };
  ```
  <br/>



#### `const uint8_t CURRENT_FORK_INDEX = FORK_HEIGHTS_SIZE == 0 ? 0 : 3;`<br/>

Este valor se relaciona con la matriz anterior FORK_HEIGHTS.
Determina qué alturas de bifurcación admite el software. (Este valor tiene índice cero).
Por ejemplo, si nuestro `FORK_HEIGHTS = {100, 200, 300}` and `CURRENT_FORK_INDEX = 1`
luego el soporte de la bifurcación en FORK_HEIGHTS [1] - que es la bifurcación en 200 bloques.

Tenemos un ternario para verificar FORK_HEIGHTS_SIZE, por lo tanto, si desea vaciar la matriz FORK_HEIGHTS,
no necesita establecer un CURRENT_FORK_INDEX.

- `const uint8_t CURRENT_FORK_INDEX = FORK_HEIGHTS_SIZE == 0 ? 0 : 1;`<br/>




#### `const char CRYPTONOTE_NAME[] = "TurtleCoin";`<br/>

Esto es obvio. Es el nombre de tu moneda!

- `const char CRYPTONOTE_NAME[] = "SuperCoolCoin";`<br/>





#### `const int P2P_DEFAULT_PORT` and `const int RPC_DEFAULT_PORT`

Estos valores definen los puertos que usa el servicio para comunicarse.
No existe un problema real con el uso de los mismos valores que el software TurtleCoin,
pero si está ejecutando tanto un servicio TurtleCoin como su propio servicio, debería 
cambiar el puerto que utiliza, ya que ambos necesitarán control sobre el puerto.

Los puertos pueden estar en el rango de 0 - 65535, sin embargo, no se recomienda el uso de puertos en 
el rango 0 - 1023, ya que en los sistemas Unix un programa requerirá permisos administrativos para usar
estos puertos.

- `const int P2P_DEFAULT_PORT = 10101;`
- `const int RPC_DEFAULT_PORT = 10102;`
<br/>




#### `const static boost::uuids::uuid CRYPTONOTE_NETWORK =`

Este valor establece la 'ID de red' de su moneda. Esto garantiza que su red
no responderá a las conexiones de red de otras monedas, como TurtleCoin.

Este valor está usando valores hexadecimales, que deberían estar en la forma 0x?? donde `?` es
un valor hexadecimal válido (0-9, a-f).

- ```
  const static boost::uuids::uuid CRYPTONOTE_NETWORK =
  {
      {  0x11, 0x22, 0x33, 0x44, 0x55, 0x66, 0x77, 0x88, 0x99, 0xaa, 0xbb, 0xcc, 0xdd, 0xee, 0xff, 0x00  }
  };
  ```
<br/>



#### `const char* const SEED_NODES[] = {`<br/>

Esta variable define los nodos de inicialización a los que los servicios se conectarán desde el primer lanzamiento,
antes de que se den cuenta de que hay pares. Tendrá que ejecutar estos en diferentes servidores 
para arrancar otros nodos. Una vez que se establezca su moneda,
podría ser una buena idea pedirle a otros miembros de la comunidad de confianza que alojen algunos nodos semilla,
para ayudar a descentralizar su moneda.

- ```
  const char* const SEED_NODES[] =
  {
      "111.111.111.111:11897",
      "222.222.222.222:11897",
  };
  ```
  <br/>




## Pre-minado

Es posible que hayas notado que nos salteamos

```
const uint64_t GENESIS_BLOCK_REWARD = UINT64_C(0);
```

y

```
const char GENESIS_COINBASE_TX_HEX[] = "010a01ff000188f3b501029b2e4c0281c0b02e7c53291a94d1d0cbff8883f8024f5142ee494ffbbd088071210142694232c5b04151d9e4c27d31ec7a68ea568b19488cfcb422659a07a0e44dd5";
```

Quería poner estos en una sección diferente,
porque la generación de un preminismo requiere un poco más de esfuerzo que los ajustes de configuración anteriores.

Para empezar, poner su cantidad en premine `const uint64_t GENESIS_BLOCK_REWARD = UINT64_C(0);`,

Por ejemplo:

- `const uint64_t GENESIS_BLOCK_REWARD = UINT64_C(100);`

Como siempre, este valor está en *unidades atómicas*. El código anterior precedería a 1 TRTL, asumiendo que se están usando 2 decimales.

A continuación, compila tu código.

Una vez que se haya compilado, use zedwallet para generar una dirección. No se preocupe por ejecutar el servicio, no lo necesitamos aún.

Guarde esta dirección en alguna parte, y asegúrese de guardar sus llaves de billetera en un lugar seguro también.

Ahora, tome esta dirección y ejecute su servicio con los siguientes argumentos:

`--print-genesis-tx --genesis-block-reward-address <premine wallet address>`

Reemplazando con la dirección que acaba de generar.

Por ejemplo:

```
TurtleCoind --print-genesis-tx --genesis-block-reward-address TRTLv2Fyavy8CXG8BPEbNeCHFZ1fuDCYCZ3vW5H5LXN4K2M2MHUpTENip9bbavpHvvPwb4NDkBWrNgURAd5DB38FHXWZyoBh4wW
```

Esto imprimirá un hash largo. Toma este hash y reemplaza el valor de

`const char GENESIS_COINBASE_TX_HEX[] =` con este valor.

- ```
  const char GENESIS_COINBASE_TX_HEX[] = "012801ff000100023f6cd7a559d3b5d88fc6396a7a261c70a5212608bdeeb73ac53ff5dad157326221015bf857a5c95d6066c23800a6563e79acc85d450d7563cf68995d1c5c92b1567a";
  ```

Ahora, recompile su código y configure sus nodos semilla.
Una vez que comience a extraer, lo preminado debería aparecer en la billetera que creó anteriormente.
Asegúrate de tener tus llaves privadas para restaurar esta billetera.

## WalletConfig.h

El siguiente paso para modificar es WalletConfig.h, ubicado en `src/zedwallet/WalletConfig.h`

Estos campos ya están bastante bien documentados, pero los revisaremos de todos modos.<br/>

#### `const std::string addressPrefix = "TRTL";`<br/>

Este valor se usa para verificar que las direcciones ingresadas sean correctas.
Este valor corresponde al valor que eligió para su prefijo de dirección anteriormente,
en `const uint64_t CRYPTONOTE_PUBLIC_ADDRESS_BASE58_PREFIX =`

- `const std::string addressPrefix = "aPPLE";`<br/>




#### `const std::string ticker = "TRTL";`<br/>

Esto se refiere al "nombre corto" que tiene su moneda, que a menudo se utiliza como un ticker en las casas de cambio. 
Por ejemplo, en TurtleCoin esto es TRTL, en Monero esto es XMR, y en Bitcoin esto es BTC.

- `const std::string ticker = "APPLE";`<br/>




#### `const std::string daemonName = "TurtleCoind";`<br/>

Esta variable determina cuál es el nombre de tu servicio.
Hablaremos sobre cambiar los nombres de los ejecutables generados en la sección `CmakeLists.txt`.
Vamos a omitir mencionar `walletName`, y `walletdName` ya que ambos siguen el mismo formato.

- `const std::string daemonName = "AppleCoind";`<br/>



#### `const std::string contactLink = "http://chat.turtlecoin.lol";`<br/>

Este valor se usa para informar al usuario dónde pueden obtener asistencia si su billetera se bloquea durante la sincronización. En nuestro caso, esta es la discordia TurtleCoin. ¿Quizás tengas un foro o un chat de IRC?

- `const std::string contactLink = "https://applecoin.com/livechat"`<br/>




#### `const long unsigned int standardAddressLength = 99;`

Este valor se usa para verificar que las direcciones ingresadas sean correctas.
Puede obtener este valor fácilmente compilando, generando una dirección con zedwallet y verificando cuánto tiempo es.


- `const long unsigned int addressLength = 100;`<br/>





#### `const long unsigned int integratedAddressLength = 236;`<br/>

Este valor se usa para verificar que las direcciones ingresadas sean correctas.
Una dirección integrada es una dirección que también contiene una identificación de pago integrada,
para evitar que los usuarios tengan que recordar proporcionar una con la transacción.

Puede obtener este valor fácilmente compilando, generando una dirección con zedwallet,
luego usando el comando `make_integrated_address`, y verificando cuánto tiempo es.

Puede usar su propia dirección con este comando y cualquier ID de pago.
Si no sabe cómo generar una identificación de pago, aquí hay una al azar para que la use.

`8eba031ca60bf9b9f680309819bddf071e619c53ff71766e48e365812e229452`

- `const long unsigned int integratedAddressLength = 237;`<br/>



#### `const uint64_t defaultMixin = CryptoNote::parameters::DEFAULT_MIXIN;`<br/>

Esto establece el valor de mixin para ser utilizado con las transacciones.
Asegúrate de que esté dentro de los límites establecidos anteriormente, con `MINIMUM_MIXIN_V1/V2` y `MAXIMUM_MIXIN_V1/V2`.


- `const uint64_t defaultMixin = 7;`<br/>




#### `const uint64_t defaultFee = CryptoNote::parameters::MINIMUM_FEE`<br/>

Si desea establecer una tarifa predeterminada más alta, tal vez para que las transacciones se realicen más rápido, puede
hacerlo aquí.
Como de costumbre, esto es en *unidades atómicas*.
También es posible que desee cambiar `const uint64_t minimumFee = CryptoNote::parameters::MINIMUM_FEE;`
para evitar que los usuarios envíen una tarifa más baja de lo deseado.

Recuerde que este límite solo se aplica con zedwallet, y el usuario puede cambiar el límite y volver a compilar,
o utilizar un programa de billetera diferente.
Si desea aplicar tarifas más altas a través de la red,
cambie el valor de `CryptoNote::parameters::MINIMUM_FEE` en `CryptoNoteConfig.h`.


- `const uint64_t defaultFee = 100;`<br/>




#### `const bool mixinZeroDisabled = true;`<br/>

¿Se permite un mixin de cero en la red? Si en algún punto se desactiva una mixin de cero,
pero ese punto está en una bifurcación posterior, aún establezca esto en `true`.

Si establece esto en verdadero, puede cambiar el valor de

```
const uint64_t mixinZeroDisabledHeight = CryptoNote::parameters::MIXIN_LIMITS_V2_HEIGHT;
```

Para determinar a qué altura de bloque se desactiva una mixin de cero.
En el caso de TurtleCoin, esto se deshabilitó en el bloque 620k,
pero al ser una red nueva, tiene la ventaja de poder deshabilitar una mezcla de cero mucho antes,
o incluso desde el lanzamiento de su red.

De nuevo, esto es solo una limitación del lado del cliente.
Asegúrese de haber establecido sus límites de mixin `CryptoNoteConfig.h` para que la red aplique los límites de mixin. 
Cualquier idea que siempre desee que suceda, debe hacerse a nivel de red; de lo contrario, se pueden escribir servicios / billeteras, etc. para evitar esas ideas / límites.

Si quiere permitir cero mixins, entonces `mixinZeroDisabledHeight` no hace nada.

- ```
  const bool mixinZeroDisabled = true;
  const uint64_t mixinZeroDisabledHeight = 0;
  ```
  <br/>






## Preparado

Ok, entonces terminaste de ajustar todas las configuraciones y ¡ya estás listo!
Para empezar, deberá compilar su código. Vea la sección de compilación a continuación.

Asegúrate de que tienes las direcciones IP de tus nodos semilla en la configuración,
3 servicios deben estar conectados a la red para que funcione tu moneda.

Una vez que haya compilado el código en todas sus máquinas,
puede iniciar su servicio.
Si no puede conectarse a los nodos de semilla,
verá algo parecido a `Failed to connect to seed nodes` a la salida de su servicio.

Asegúrese de que sus firewalls no impidan la conexión del servicio,
puede que necesite abrir los puertos p2p y RPC para el servicio, que por defecto son 11898 y 11897.

Si tus servicios ahora están hablando entre ellos, ¡puedes comenzar a extraer, y lanzar tu blockchain!

Puede usar el `miner` ejecutable de minería individual para esto.
FDesde un indicador de comando o terminal, inicie el minero de esa manera: `miner --threads 1 --log-level 3 --address <your address>`,
reemplazando <your address> con una dirección que haya generado y controlado.

Por supuesto, puede aumentar el valor de los subprocesos, sin embargo, dado que usted es el único que está minando actualmente en la red, ¡aún no tiene mucho sentido!

Por ejemplo:

```
./miner --threads 8 --log-level 3 --address TRTLuyTvsJZjAsbtgbYmZeV1pb43dXqcv1jNdYNwokv8GUa1ZbYzivzg2gEgXeAfUqJF12APf3Rq89UneaQKiZ1nGW1vYqkGb8Y
```

Si ha hecho esto correctamente, debería ver resultados como este:
`Block successfully submitted, hash xxxx`, y en su ventana de servicio, debería ver el
debería ver el valor `New block detected` aumentando en altura.

¡Felicidades!

## Compilando

Siempre puede encontrar las últimas instrucciones sobre cómo compilar TurtleCoin en nuestro [GitHub](https://github.com/turtlecoin/turtlecoin#how-to-compile).


## Soporte, ¿Preguntas?

Algo no está claro? Dirígete a nuestro [Discord](http://chat.turtlecoin.lol) y pregunta en el canal #help,
y con suerte, alguien podrá ayudarte.

Háganos saber si algo está mal con esta guía, o falta, para que podamos actualizarla y mejorarla.

¡Buena suerte con tu nueva moneda!
