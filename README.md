# AWS-GITHUB-CODECOMMIT-EC2-CODEDEPLOY
Este repositorio es una replica del tutorial de (https://docs.aws.amazon.com/codepipeline/latest/userguide/tutorials-simple-codecommit.html) pero en español además de con unas explicaciones más extendidas.

# Objetivo del repositorio

Se usa CodePipeline como herramienta para poner en producción el código de CodeCommit y tener el resultado reflejado en un EC2. 

El desencadenador será el push que se haga al CodeCommit como cambio del repositorio. 

## 1. Creación del repositorio de CodeCommit

CodeCommit funciona igual igual que Github/Bitbucket u otras herramientas que pueden haber en el mercado para gestionar las versiones de código. 

1. Seleccionamos la region donde queremos desarrollar nuestro proyecto dentro de CodeCommit. 

2. Seleccionamos "CREATE REPOSITORY". 

3. En el nombre indicamos:

```
MyDemoRepo
```
4. Creamos el repositorio

### Gestionamos el código de ESTE Github para subirlo a Codecommit. 

1. Abrimos una termina Git y descargamos el repositorio de AWS: 

```
https://git-codecommit.us-east-1.amazonaws.com/v1/repos/MyDemoRepo
```

2. Clonamos ESTE repositorio tambien


```
https://github.com/KAIET98/AWS-GITHUB-CODECOMMIT-EC2-CODEDEPLOY.git
```
3. Copiamos y pegamos los archivos de ESTE repositorio al de AWS y lo subimos 

```
git add -A
git commit -m "Subo los archivos del proyecto"
git push

```

## 2. Creamos el EC2 para ver reflejado el output

### Gestión de Roles

Para poder hacer un deploy en el EC2 necesitamos tener un acceso, es decir un permiso en este caso es el: 

```
AmazonEC2RoleforAWSCodeDeploy
```
Entonces

1. Abrimos IAM

2. Roles

3. Le damos a "Create Role"

4. En los servicios de AWS seleccionamos "EC2".

5. Nombre del policy

```
AmazonEC2RoleforAWSCodeDeploy
```

6. Al rol le daremos el siguiente nombre:

```
EC2InstanceRole
```


## Lanzamos el EC2

1. Selecionamos la instancia mínima para que no nos carguen gastos adicionales, aparecerá un "free tier eligible".

```
Amazon Linux 2 AMI (HVM), SSD Volume Type
```

2. El tipo de instancia también será la mínima

```
t2.micro
```

3. Configuramos los detalles de la Instancia en "Next: Configure Instance Details"

4. Numero de instancias = 1

5. Auto-assign Public IP = "ENABLE". 

6. IAM Role, vamos a seleccionar el que acabamos de crear:

```
EC2InstanceRole
```
7. Y en detalles avanzados pegaremos los siguientes comandos.

```
#!/bin/bash
yum -y update
yum install -y ruby
yum install -y aws-cli
cd /home/ec2-user
wget https://aws-codedeploy-us-east-2.s3.us-east-2.amazonaws.com/latest/install
chmod +x ./install
./install auto
```
Con este código el CodeDeploy se instala el agente.

8. En el apartado de Add Tag seleccionamos el key = a "Name" y el value = 

```
MyCodePipelineDemo
```

Ahora no es relevante este último detalle pero luego a la hora de crear el CodePipeline tiene mucho valor.

9. En "Configure Security Group", se añadiremos un grupo de seguridad mediante: Create a new security group

- En SSH, elegiremos nuestro IP. 

Y en la Add Rule, escogeremos "HTTP" y tambien elegiremos nuestra IP.

10. Lanzamos la instancia. 

## 3. Creamos la aplicacion en CodeDeploy

### Gestión de roles

Igual que nos ha pasado con el EC2, aquí también tenemos que ejecutar unas roles antes de hacer nada con nuestro usuario. 

1. Accedemos a IAM>Roles
2. Creamos un rol
3. En AWS Servicios escogemos el "CodeDeploy" y en permisos escogemos el: 


```
AWSCodeDeployRole
```
4. Como nombre al rol le ponemos: 

```
CodeDeployRole
```

### Creación de aplicación en CodeDeploy

1. En Applications, seleccionamos "Create Application" y como nombre le indicamos:

```
MyDemoApplication
```

2. Como Compute Platform indicaremos que sea el "EC2/On-premises", y creamos la aplicación. 

### Creación de grupo de producción en CodeDeploy

El grupo nos da la información de como desarrollar nuestra aplicación y en qué tipo de instancias lo podemos desplegarlo.

Ahora sí que tiene sentido el Value Tag creado en EC2.

1. Le seleccionamos el "Create deployment group". 

2. En el nombre indicamos:

```
MyDemoDeploymentGroup
```

3. En los roles le vamos ai indicar que se referencie a uno que hemos creado antes, y seleccionamos el 

```
CodeDeployRole

```
que aparecia en  3. Creamos la aplicacion en CodeDeploy/Creación de aplicación en CodeDeploy


4. Seguido en Deployment Type, indicamos "In-place". 

5. Environment configuration, seleccionamos el: 

```
Amazon EC2 Instances
```
Dando como valor al Key = "Name" y en el Value:

```
MyCodePipelineDemo
```

6. En environment Configuration seleccionamos el: 

```
CodeDeployDefault.OneAtaTime
```

7. Desactivamos el Load Balancer como las Alarmas dentro de Advanced.


### Creación de pipeline en CodePipeline


1. Seleccionamos CreatePipeline

2. STEP1: En el nombre indicaremos: 


```
MyFirstPipeline
```

3. En los Roles, le indicaremos que: 

```
New service role
```

4. En servicio Avanzados, los dejamos como tal.

5. STEP.2: En Source Provider, le vamos a decir que lea le código por medio de CodeCommit. 

Los ajustes por defecto los dejamos por defecto. 

6. STEP3: Add deploy stage: 

Como entorno de desarrollo, le indicamos que lo haga sobre CodeDeploy y que la aplicación se llame: 


```
MyDemoApplication
```

El que le hemos indicado antes arriba vamos.

7. En Deployment groups, indicaremos
```
MyDemoDeploymentGroup
```

y finalmente le damos a "Create Pipeline".


## Verificación del output

Si todo se ha hecho correctamente, en EC2, seleccionamos la VM que hemos creado en este tutorial y en la descripción en DNS Público, tendremos una dirección de este estilo:



```
ec2-44-202-196-193.compute-1.amazonaws.com
```

Si abrimos una ventana y pegamos al URL deberia de salirnos el contenido del fichero HTML del repositorio.