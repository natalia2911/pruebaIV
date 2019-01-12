# Despliegue en Azure

Para hacer el despliegue en Azure lo primero que tenemos que hacer es crearnos una cuenta en Azure, y usar los créditos que el profesor de la asignatura nos ha proporcionado para usar.

Instalamos el cliente de Azure:

`	curl -L https://aka.ms/InstallAzureCli | bash`

Nos logeamos :  
`az login`

![](https://github.com/natalia2911/ProyectoIV-BOT/blob/master/img/login.png)

##  Máquina Virtual: Vagrant

Para crear y desplegar nuestra aplicación necesitaremos una máquina virtual, en este caso hemos utilizado Vagrant con un plugin de Azure para ello.

Para la creación de la máquina, necesariamente tendremos que crear un archivo  [Vagrantfile](https://github.com/natalia2911/ProyectoIV-BOT/blob/master/Vagrantfile)

Creamos el archivo **Vagrantfile** :
```
vagrant init
```

Este es el archivo :
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  # Box que vamos a utilizar en la máquina virtual.
  config.vm.box = "noticieroapp"
  # Especificamos el dummy box, el cual nos proporcionará una base para nuestra máquina.
  config.vm.box_url = 'https://github.com/msopentech/vagrant-azure/raw/master/dummy.box'

  # Le especificamos la clave, para la conexión mediante ssh de la máquina.
	config.ssh.private_key_path = "~/.ssh/id_rsa"

  # Configuración de la máquina (proveedor) donde vamos a crear el host de la máquina:
	config.vm.provider :azure do |noticiero, override|

        # Variables de entorno y parametros, necesarios para crear nuestra máquina con Azure.
    		noticiero.tenant_id = ENV['AZURE_TENANT_ID']  # Tenant id
    		noticiero.client_id = ENV['AZURE_CLIENT_ID'] #Id del cliente de azure
    		noticiero.client_secret = ENV['AZURE_CLIENT_SECRET']  #Contraseña de del cliente de azure
    		noticiero.subscription_id = ENV['AZURE_SUBSCRIPTION_ID']  #Subscripción de Azure.

        # Tamaño de los recursos de Azure
		    noticiero.vm_size = "Standard_A0"
        # Localización donde queremos el host, o la granja de servidores donde
        # nos asignan la máquina
		    noticiero.location = "westeurope"
        #Abrimos el puerto 80, que es el que vamos a utilizar
		    noticiero.tcp_endpoints = "80"
        #Nombre de la máquina virtual
		    noticiero.vm_name = "noticieroapp"
        #Especificamos la imagen que vamos a montar en nuestra máquina, en este caso Ubuntu 16.04
        noticiero.vm_image_urn = 'Canonical:UbuntuServer:16.04-LTS:latest'
        #Grupo de recursos en los que se creará la máquina.
        noticiero.resource_group_name = 'ProyectoNoticiero'
  end

  # Configuración del provisionamiento con ansible, con nuestro fichero de playbook.yml
  # Durante el vagrant up, nos permite que el fichero playbook.yml nos instale las dependencias que necesita
  # nuestra máquina
	config.vm.provision :ansible do |ansible|
		ansible.playbook = "./provision/playbook.yml"
	end
end
```
Vamos a proceder a explicar cada uno de los diferentes parametros de configuración que hemos incluido en nuestro Vagrantfile:

- **config.vm.box** : que nos especifica el box que vamos a utilizar en la máquina virtual; tendremos que elegir uno predefinido o crear uno.
```
vagrant box add noticieroapp https://github.com/azure/vagrant-azure/raw/v2.0/dummy.box --provider azure
```
![](https://github.com/natalia2911/ProyectoIV-BOT/blob/master/img/boxvagrant.png)

- **config.vm.box_url**: especificamos el dummy box que nos proporcionará una base para nuestra máquina virtual.
- **config.ssh.private_key_path**: especificamos la clave para la conexión mediante ssh de la máquina.
- **config.vm.provider :azure do |noticiero, override|**: Aquí vamos a proceder a indicar la configuración de la máquina, o el proveedor, donde vamos a crear el host de nuestra máquina virtual.  Especificamos con un string el nombre del proveedor en nuestro caso *Azure* y las dos variables que tenemos que son *noticiero* para hacer referencia a los diferentes parametros de configuración, y *override* la cual es necesaria para sobreescribir la aplicación.
- (Dentro de la configuración mencionada anteriormente tenemos) -->

		noticiero.tenant_id = ENV['AZURE_TENANT_ID']  
	    noticiero.client_id = ENV['AZURE_CLIENT_ID']
	    noticiero.client_secret = ENV['AZURE_CLIENT_SECRET']  
	    noticiero.subscription_id = ENV['AZURE_SUBSCRIPTION_ID']  

	Variables de entorno y parámetros, necesarios para crear nuestra máquina con Azure.

	Para obtener los tres primeros datos ejecutaremos
	```
	az ad sp create-for-rbac
	```
	Por terminal nos aparecerá algo así:
![](https://github.com/natalia2911/ProyectoIV-BOT/blob/master/img/datos.png)

	Para obtener la suscripción :
	```
	az account list --query "[?isDefault].id" -o tsv
	```
![](https://github.com/natalia2911/ProyectoIV-BOT/blob/master/img/subscripcion.png)

-	**noticiero.vm_size**: tamaño de los recursos de Azure, en nuestro caso hemos elegido *Standard_A0* ya que es uno de los más básicos y báratos, y se adapta a nuestra necesidades. Tiene 1 core, 0.75 GiB y cuesta 0.02$/hora. Tenemos más aquí : https://azureprice.net/ los podemos elegir también dependiendo de la localización de las granjas.
-	**noticiero.location**: localización donde queremos el host, o la granja de servidores donde nos van a asignar la máquina.
- **noticiero.tcp_endpoints** : especificamos el puerto 80, que es el que abrimos, por que de momento, solo vamos a usar este.
-	**noticiero.vm_name**: nombre de la máquina virtual.
-	**noticiero.vm_image_urn**: especificamos la imagen que vamos a montar en nuestra máquina, en nuestro caso vamos a usar Ubuntu 16.04
-	**noticiero.resource_group_name**: grupo de recursos en los que se creará la máquina
-	**config.vm.provision :ansible do |ansible|**: Configuración del provisionamiento con ansible, con nuestro fichero de playbook.yml, Durante el vagrant up, nos permite que el fichero playbook.yml nos instale las dependencias que necesita nuestra máquina, hemos usado; Especificamos con un string el nombre de la provisión en nuestro caso _Ansible_ y las variable para el playbook *Ansible*.

Un apunte que debemos realizar es que anteriormente nos hemos estado refiriendo a *config* ya que para la configuración de vagrant hemos usamos esta varible y lo especificamos en esta linea: **Vagrant.configure("2") do |config|**

Y ya podemos iniciar la máquina virtual:
```
vagrant up --provider=azure
```


## Provisionamiento: Ansible

Crearemos el archivo de provisionamiento para la máquina, donde tendremos todo lo que esta necesitará, en nuestro caso se llamará `playbook.yml`




En este pasó se ejecutará el fichero de provisionamiento, pero antes deberemos añadir nuestro host  en el fichero : `/etc/ansible/hosts`
podremos poner, tanto nuestra IP de la máquina (la cual miramos en Azure Portal) o nuestra dirección en nuestro caso: `noticieroapp.westeurope.cloudapp.azure.com `

En el caso en que nuestro **playbook.yml** no se haya ejecutado a la hora de crear la máquina, lo podremos hacer desde terminal con esta orden:

`ansible-playbook provision/playbook.yml
`


## Despliege : Fabfile

 Para hacer el despliegue tendremos que crear el archivo `fabfile.py` que realizará una serie de ordenes de forma remota mediante ssh.
 Las tareas que puede realizar es conectarse al servidor remoto, actualizar el repositorio, borrar los datos del código antiguo..

Las funciones que hemos definido son:

 - Desinstalar: borra todo el repositorio, el código anterior
 - Instalar: se instala la aplicación
 - Iniciar: lanza la aplicación y la ejecuta en segundo plano.
 -
![](https://github.com/natalia2911/ProyectoIV-BOT/blob/master/img/desinstalar.png)
![](https://github.com/natalia2911/ProyectoIV-BOT/blob/master/img/instalar.png)
![](https://github.com/natalia2911/ProyectoIV-BOT/blob/master/img/iniciar.png)

Para poder probar las funciones del despliegue:

`fab -f despliegue/fabfile.py -H vagrant@noticieroapp.westeurope.cloudapp.azure.com <Iniciar/Instalar/Desinstalar>`

Por último, el despliegue final lo hemos realizado:

`IP : 13.80.251.123`

`DNS : noticieroapp.westeurope.cloudapp.azure.com`

Podemos comprobar que ahora la maquina virtual está efectivamente ejecutando el servicio:

![](https://github.com/natalia2911/ProyectoIV-BOT/blob/master/img/ip.png)
![](https://github.com/natalia2911/ProyectoIV-BOT/blob/master/img/dns.png)

Aquí me gustaría hacer un apunte sobre nuestro fichero **fabfile**, en el podríamos haber puesto la ejecución de los test de la siguiente manera:

    def test():
        with cd("ProyectoIV-BOT/test/"):
            result = run("python3 test.py")
        if result.failed and not confirm("Tests failed. Continue anyway?"):
            abort("Aborting at user request.")

Pero hemos decidido que esta parte la dejaremos para próximas mejoras, esto lo hemos conocido consultado la documentación concretamente de la página : http://docs.fabfile.org/en/1.14/tutorial.html

Aquí adjuntamos los enlaces a la documentación que hemos usado para crear y desplegar nuestra aplicación:

**Fabfile**
https://moduslaborandi.net/post/introduccion-a-fabric-ii/
https://axiacore.com/blog/sudo-y-fabric-para-administrar-y-desplegar-como-devop/
https://blocknitive.com/blog/workshop-hyperledger-fabric-despliegue-de-una-red-fabric-para-universidades/
https://docs.fabfile.org/en/1.4.3/
http://docs.fabfile.org/en/1.14/tutorial.html

**Vagrantfile**
https://www.rubydoc.info/gems/vagrant-azure/1.3.0

**Ansible : Playbook**
https://blog.deiser.com/es/primeros-pasos-con-ansible
https://stackoverrun.com/es/q/7574013
