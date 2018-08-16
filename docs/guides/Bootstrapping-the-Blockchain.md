# Cómo aplicar Bootstrap a la cadena de bloques de TurtleCoin

Esta guía te ayudará a instalar una copia reciente de la cadena de bloques. Esto debería acelerar significativamente la tarea de sincronizar el cadena de bloques para que pueda usar su billetera.

## Windows:
1. Asegúrese de que `TurtleCoind.exe`, `walletd.exe`, y / o la billetera GUI no se estén ejecutando.

2. Abra File Explorer y escriba `%APPDATA%\TurtleCoin` y presione enter.

    ![file explorer](images/bootstrap/file_explorer.jpg)

    !!! Nota
        Si la carpeta no existe, solo ve a ella `%APPDATA%` y crea una carpeta con el nombre `TurtleCoin`.


3. Elimine los siguientes si existen:
    * blockindexes.bin
    * blocks.bin
    * "DB" folder


    ---
    
    **Nota**: En caso de que no pueda eliminar los archivos debido a que son utilizados por algún otro programa, siga estos         pasos:
    
    * Abra el Administrador de tareas con el acceso directo `Ctrl + Shift+ Escape`.
      
    * Haga clic en `Processes`.
      
    * Haga clic en `Image Name`.
      
    * Desplázate hacia abajo.
      
    * Haga clic en `walletd.exe`
      
    * Haga clic en `End Process`.
      
    * Haga clic en `End Process` nuevamente.
      
    * Intenta borrarlos de nuevo.
    
    ![closewallet](images/bootstrap/close_walletd.png)
    
    ---


4. [Descargue](https://f000.backblazeb2.com/file/turtle-blockchain/latest.zip) la última instantánea de la cadena de bloques.

5. Mueva los dos nuevos archivos descargados a la `%APPDATA%\TurtleCoin` carpeta.

6. Comience `TurtleCoind.exe` o la billetera GUI como lo hace normalmente.

7. Vea la sección de [Resultados Esperados](#ExpectedResults) a continuación.



## Mac & Linux:
1. Asegúrese de que `TurtleCoind`, `walletd`, y / o la billetera GUI no se estén ejecutando.

2. Abrá `Finder`.

3. Utilice el atajo `Command + Shift + G` para que aparezca `Go to Folder`:

    ![findergoto.jpg](images/bootstrap/findergoto.jpg)

4. Elimine los siguientes si existen:

    * blockindexes.bin 

    * blocks.bin 

    * "DB" folder 


5. [Descargue](https://f000.backblazeb2.com/file/turtle-blockchain/latest.zip) la última instantánea de la cadena de bloques.

6. Mueva los dos nuevos archivos descargados, `blockindexes.bin` y `blocks.bin` dentro de la `~/.TurtleCoin/` carpeta.

7. Comience `TurtleCoind` o la GUI como lo hace normalmente.

8. Vea la sección de [Resultados Esperados](#ExpectedResults) a continuación.

## Resultados esperados si se hace correctamente <a name="ExpectedResults"></a>

Cuando comiences `TurtleCoind` you should see this. deberías ver esto. Tenga en cuenta que el tamaño de bloques (150246) en este ejemplo será un número diferente:
```
2018-Feb-01 18:43:37.216471 INFO    Initializing core...
2018-Feb-01 18:43:37.225492 INFO    Importing blocks from blockchain storage
2018-Feb-01 18:43:37.741587 INFO    Imported block with index 1000 / 150246
2018-Feb-01 18:43:38.258202 INFO    Imported block with index 2000 / 150246
2018-Feb-01 18:43:38.928033 INFO    Imported block with index 3000 / 150246
2018-Feb-01 18:43:39.454094 INFO    Imported block with index 4000 / 150246
2018-Feb-01 18:43:40.142969 INFO    Imported block with index 5000 / 150246
2018-Feb-01 18:43:40.830674 INFO    Imported block with index 6000 / 150246
```

Después de que se complete, comenzará a sincronizarse incrementalmente de esta manera:
```
2018-Feb-01 18:47:48.075930 INFO    Imported block with index 150000 / 150246
2018-Feb-01 18:47:48.860470 INFO    Core initialized OK
2018-Feb-01 18:47:48.860470 INFO    Initializing p2p server...
```

Si está utilizando la billetera GUI, puede ver el progreso abriendo `walletd.log` y desplazándose hacia la parte inferior.
