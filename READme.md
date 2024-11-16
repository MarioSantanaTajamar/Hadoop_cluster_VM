
# Levantar un clúster con Hadoop en VM
Ejecutar los comandos en el orden que aparecen
## 1.-Descargar la imágen

Ubuntu server 24.04: https://www.osboxes.org/ubuntu-server/

## 2.-Montar la VM con la imagen
Set up recomendado:

-Número de procesadores: 1

-RAM 2560

-Disco duro: 25 GB

-Network: puente (para poder usar ssh)

## 3.-Encender la VM

¡¡RECORDAR INSTALAR SSH EN LA CONFIGURACIÓN SI QUEREMOS USAR PS!!

Para conectarnos por ssh usando PowerShell
```bash
  ssh <usuario>@<ip_de_la_máquina>
```

## 4.-Crear un administrador

```bash
  sudo adduser hduser
```
```bash
  sudo usermod -aG sudo hduser
```
```bash
  exit
```

## 5.-Hacer login con "hduser", descargar y actualizar paquetes necesarios

```bash
  sudo apt update
```
```bash
  sudo apt install vim ssh net-tools openjdk-11-jdk git
```
```bash
  cd ~
```
```bash
  wget https://archive.apache.org/dist/hadoop/common/hadoop-3.3.2/hadoop-3.3.2.tar.gz
```
```bash
  tar xvf hadoop-3.3.2.tar.gz
```

Este comando (reboot) hay que hacerlo en el terminal de la máquina. 
```bash
  init 0
```

## 6.-Habilitar administrador de red
 6.1.-En archivo > Herramientas > administrador de red
 
![Cambiar administrador de red](/img/1.png)

 6.2.-Habilitar el servidor DHCP

![Habilitar DHCP](/img/2.png)

 ## 7.-Configuración de la red de la VM
¡¡La máquina debe estar apagada!!

 Configuración > Red > Habilitar adaptador red

Poner el adaptador red en "sólo anfitrión"

![Adaptador red](/img/3.png)

## 8.-Arrancar máquina y cambiar a ip estática
(Hacer login como hduser)

```bash
sudo ip a
```
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

Pegar esto en el archivo "01-netcfg.yaml":
```bash
  network:
    version: 2
    renderer: networkd
    ethernets:
        enp0s3:
            dhcp4: true
        enp0s8:
            addresses: [192.168.56.50/24]
            dhcp4: false
```

```bash
sudo netplan apply
```
```bash
sudo ip a
```
```bash
sudo nano /etc/hosts
```
Pegar esto en hosts:
```bash
192.168.56.50   master
192.168.56.51   worker1
192.168.56.52   worker2
192.168.56.53   worker3
```
![Donde pongo el texto](/img/4.png)

```bash
sudo hostnamectl set-hostname master
```

## 9.-Configurar Hadoop

```bash
cd ~
```

```bash
nano ~/.bashrc
```

```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_HOME=$HOME/hadoop-3.3.2
export PATH=$HADOOP_HOME/sbin:$HADOOP_HOME/bin:$PATH
```

![Exports](/img/5.png)

```bash
source ~/.bashrc
```
```bash
git clone https://github.com/nilesh-g/hadoop-cluster-install.git
```

```bash
sudo cp hadoop-cluster-install/master/* $HADOOP_HOME/etc/hadoop/
```

Verificar que se ha copiado todo bien
```bash
cd $HADOOP_HOME/etc/hadoop/
```
```bash
ls
```

## 10.-Crear otra máquina "worker1" como clon enlazado

Para esto tenemos que tener apagada la máquina.

Clonar la VM

![Como clonar](/img/6.png)

- Nombre: worker1
- Clonación: enlazada
- Política: Generar nuevas direcciones...

![Configuracion VM](/img/7.png)

## 11.-Configurar la red y hadoop en worker1
Iniciar sesión como "hduser"

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
Cambiar el adress a 51 (copiar esto)
```bash
network:
    version: 2
    renderer: networkd
    ethernets:
        enp0s3:
            dhcp4: true
        enp0s8:
            addresses: [192.168.56.51/24]
            dhcp4: false
```

![IP](/img/8.png)

```bash
sudo netplan apply
```
```bash
sudo ip a
```
```bash
sudo hostnamectl set-hostname worker1
```
```bash
cd ~
``` 
```bash
sudo cp hadoop-cluster-install/worker/* $HADOOP_HOME/etc/hadoop/
```
Verificar que se ha copiado todo bien
```bash
cd $HADOOP_HOME/etc/hadoop/
```
```bash
ls
```
## 12.-Crear otra maquina igual a worker1 y llamarlo worker2 (otra copia de la VM principal)

Arrancar la máquina después de crearla

Cambiar el mismo archivo de antes pero ahora poner 52
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
```bash
network:
    version: 2
    renderer: networkd
    ethernets:
        enp0s3:
            dhcp4: true
        enp0s8:
            addresses: [192.168.56.52/24]
            dhcp4: false
```

![ip2](/img/9.png)

```bash
sudo netplan apply
```
```bash
sudo ip a
```
```bash
sudo hostnamectl set-hostname worker2
```
```bash
cd ~
```
```bash
sudo cp hadoop-cluster-install/worker/* $HADOOP_HOME/etc/hadoop/
```
El init 0 desde el terminal de ubuntu
```bash
init 0
```
## 13.-Crear un clon enlazado de worker2 como worker3 (igual que las demás de configuración)

## 14.-En worker3 hacer login con hduser, configurar hadoop y red
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
```bash
network:
    version: 2
    renderer: networkd
    ethernets:
        enp0s3:
            dhcp4: true
        enp0s8:
            addresses: [192.168.56.53/24]
            dhcp4: false
```

![ip3](/img/10.png)

```bash
sudo netplan apply
```
```bash
sudo ip a
```
```bash
sudo hostnamectl set-hostname worker3
```
```bash
cd ~
```
```bash
sudo cp hadoop-cluster-install/worker/* $HADOOP_HOME/etc/hadoop/
```

## 15.-Arrancar todas las VMs
- Vamos a copiar la ssh de master en todos los workers
En el terminal master (el original)
```bash
ssh-keygen -t rsa -P ""
```
```bash
ssh-copy-id hduser@master
```
```bash
ssh-copy-id hduser@worker1
```
```bash
ssh-copy-id hduser@worker2
```
```bash
ssh-copy-id hduser@worker3
```

## 16.-Iniciar hadoop y verificar la instalación

Aquí seguimos en el terminal de master

```bash
hdfs namenode -format
```
  - Si da error el siguiente comando porque no puede encontrar java8, seguramente tengamos el 11 (verificar versión y hacerlo con la que esté instalada) tenemos que hacer lo siguiente:
    ```bash
    nano $HADOOP_HOME/etc/hadoop/hadoop-env.sh
    ```
    - Si nos da error al guardar es por     permisos,       cambiarlos:
      ```bash
      sudo chown -R hduser:hduser /home/hduser/hadoop-3.3.2
      ```
  - La linea de JAVA_HOME borrarla y poner:
    ```bash
      export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
    ```

![export del java home](/img/11.png)

  - Guardar el archivo
  - Ahora aplicar la misma configuración a los workers:
      ```bash
    sudo scp $HADOOP_HOME/etc/hadoop/hadoop-env.sh user@worker1:$HADOOP_HOME/etc/hadoop/hadoop-env.sh
      ```
    ```bash
    sudo scp $HADOOP_HOME/etc/hadoop/hadoop-env.sh user@worker2:$HADOOP_HOME/etc/hadoop/hadoop-env.sh
    ```
    ```bash
    sudo scp $HADOOP_HOME/etc/hadoop/hadoop-env.sh user@worker3:$HADOOP_HOME/etc/hadoop/hadoop-env.sh
    ```

Seguimos con el proceso
```bash
start-dfs.sh
```
```bash
start-yarn.sh
```
```bash
jps
```
Nos movemos al terminal de cada worker y ejecutamos esto en cada uno.
```bash
jps
```

## 17.-Probar que todo vaya bien

En el terminal master
```bash
echo "Welcome to Hadoop cluster" > hello.txt
```
```bash
hadoop fs -put hello.txt /
```
```bash
hadoop fs -ls /
```
```bash
hadoop fs -cat /hello.txt
```

## 18.-Acceder a la interfaz de Hadoop para ver 


![Numero de datanodes](/img/12.png)
![File subido](/img/13.png)

## 19.-Parar Hadoop
```bash
stop-yarn.sh
```
```bash
stop-dfs.sh
```

## 20.-Apagar VMs

Si falla algun datanode probar esto
```bash
/home/hduser/hadoop-3.3.2/sbin/hadoop-daemon.sh stop datanode
```
```bash
/home/hduser/hadoop-3.3.2/sbin/hadoop-daemon.sh start datanode
``` 
En el worker específico, sirve para iniciar individualmente el datanode de ese worker, hace primero un stop por precaución
