## Function for create new laravel project

1. Este script te permite crear proyectos laravel de forma automatica.

```
function laravel() {

    echo "Por favor, seleccione un número del 1 al 8 correspondiente a la versión de PHP que desea utilizar:"

    select php_version in "5.6" "7.0" "7.1" "7.2" "7.3" "7.4" "8.0" "8.1"; do
        case $php_version in 
            "5.6" ) 
                sudo a2dismod php*
                sudo a2enmod php5.6 
                sudo update-alternatives --set php /usr/bin/php5.6 
                sudo service apache2 restart
                break;;
            "7.0" ) 
                sudo a2dismod php*
                sudo a2enmod php7.0 
                sudo update-alternatives --set php /usr/bin/php7.0 
                sudo service apache2 restart
                break;;
            "7.1" ) 
                sudo a2dismod php*
                sudo a2enmod php7.1 
                sudo update-alternatives --set php /usr/bin/php7.1 
                sudo service apache2 restart
                break;;
            "7.2" ) 
                sudo a2dismod php*
                sudo a2enmod php7.2 
                sudo update-alternatives --set php /usr/bin/php7.2 
                sudo service apache2 restart
                break;;
            "7.3" ) 
                sudo a2dismod php*
                sudo a2enmod php7.3 
                sudo update-alternatives --set php /usr/bin/php7.3 
                sudo service apache2 restart
                break;;
            "7.4" ) 
                sudo a2dismod php*
                sudo a2enmod php7.4 
                sudo update-alternatives --set php /usr/bin/php7.4 
                sudo service apache2 restart
                break;;
            "8.0" ) 
                sudo a2dismod php*
                sudo a2enmod php8.0 
                sudo update-alternatives --set php /usr/bin/php8.0 
                sudo service apache2 restart
                break;;
            "8.1" ) 
                sudo a2dismod php*
                sudo a2enmod php8.1
                sudo update-alternatives --set php /usr/bin/php8.1
                sudo service apache2 restart
                break;;
            * ) echo "Opción inválida. Seleccione una opción válida."; continue;;
        esac
    done


    echo  "Por favor, seleccione un número del 1 al 6 correspondiente a la versión de Laravel que desea utilizar:"
    select laravel_version in "10.*" "9.*" "8.*" "7.*" "6.*" "5.*"; do
        case $laravel_version in 
            10.*|9.*|8.*|7.*|6.*|5.* ) break;;
            "10.*" ) laravel_version="10.*"; break;;
            "9.*" )  laravel_version="9.*"; break;;
            "8.*" )  laravel_version="8.*"; break;;
            "7.*" )  laravel_version="7.*"; break;;
            "6.*" )  laravel_version="6.*"; break;;
            "5.*" )  laravel_version="5.*"; break;;
            *) echo "Opción inválida. Seleccione una opción válida."; continue;;
        esac
    done

    if [[ "$laravel_version" == "10.*" && "$php_version" == "5.6" ]]; then
    echo "La versión de Laravel 10.* no es compatible con PHP 5.6. Por favor, seleccione otra versión de PHP."
    exit 1
    elif [[ "$laravel_version" == "9.*" && ("$php_version" == "5.6" || "$php_version" == "7.0") ]]; then
        echo "La versión de Laravel 9.* no es compatible con PHP 5.6 ni con PHP 7.0. Por favor, seleccione otra versión de PHP."
        exit 1
    elif [[ "$laravel_version" == "8.*" && ("$php_version" == "5.6" || "$php_version" == "7.0" || "$php_version" == "7.1") ]]; then
        echo "La versión de Laravel 8.* no es compatible con PHP 5.6, PHP 7.0 ni PHP 7.1. Por favor, seleccione otra versión de PHP."
        exit 1
    fi

    read -p "Ingrese el nombre del proyecto: " project_name
    read -p "Ingrese el nombre de la base de datos: " database_name
    read -p "Ingrese el nombre del usuario de la base de datos: " database_user
    read -p "Ingrese la contraseña del usuario de la base de datos: " database_password
    read -p "Ingrese el nombre del dominio: " domain_name

    if [[ -z "$project_name" || -z "$database_name" || -z "$database_user" || -z "$database_password" || -z "$domain_name" ]]; then
        echo "Debes ingresar todos los datos requeridos."; continue;
    fi

    echo "Creando proyecto $project_name con laravel $laravel_version y php $php_version"
    
    cd /var/www/html
    composer create-project --prefer-dist laravel/laravel $project_name $laravel_version

    cd $project_name && cp .env.example .env && php artisan key:generate && chmod 777 -R storage/ && chmod 777 -R bootstrap/ && chmod 777 -R public/ && npm install

    sed -i "s/DB_DATABASE=laravel/DB_DATABASE=$database_name/g" .env
    sed -i "s/DB_USERNAME=root/DB_USERNAME=$database_user/g" .env
    sed -i "s/DB_PASSWORD=/DB_PASSWORD=$database_password/g" .env

    xdg-open http://localhost/phpmyadmin

    read -p "Presione Enter para continuar después de crear la base de datos en phpMyAdmin"
    
    php artisan migrate

    echo "Proyecto creado correctamente"

    sleep 3
    clear

    echo "Creación del entorno productivo para $laravel_version con PHP $php_version..."
    
    sudo touch /etc/apache2/sites-available/000-default.conf
    sudo bash -c "echo '<VirtualHost *:80>' >> /etc/apache2/sites-available/000-default.conf"
    sudo bash -c "echo '    ServerName $domain_name.dev.com' >> /etc/apache2/sites-available/000-default.conf"
    sudo bash -c "echo '    ServerAlias www.$domain_name.dev.com' >> /etc/apache2/sites-available/000-default.conf"
    sudo bash -c "echo '    DocumentRoot /var/www/html/$project_name/public' >> /etc/apache2/sites-available/000-default.conf"
    sudo bash -c "echo '    <Directory /var/www/html/$project_name/public>' >> /etc/apache2/sites-available/000-default.conf"
    sudo bash -c "echo '        Options Indexes FollowSymLinks MultiViews' >> /etc/apache2/sites-available/000-default.conf"
    sudo bash -c "echo '        AllowOverride All' >> /etc/apache2/sites-available/000-default.conf"
    sudo bash -c "echo '        Order allow,deny' >> /etc/apache2/sites-available/000-default.conf"
    sudo bash -c "echo '        allow from all' >> /etc/apache2/sites-available/000-default.conf"
    sudo bash -c "echo '    </Directory>' >> /etc/apache2/sites-available/000-default.conf"
    sudo bash -c "echo '    ErrorLog /var/log/apache2/$domain_name.dev.com-error.log' >> /etc/apache2/sites-available/000-default.conf"
    sudo bash -c "echo '    CustomLog /var/log/apache2/$domain_name.dev.com-access.log combined' >> /etc/apache2/sites-available/000-default.conf"
    sudo bash -c "echo '</VirtualHost>' >> /etc/apache2/sites-available/000-default.conf"
    echo -e "\n"
    sudo service apache2 restart

    sudo bash -c "echo '127.0.0.1 $domain_name.dev.com' >> /etc/hosts"
    sudo service apache2 restart

    echo "Entorno configurado correctamente"

    read -p "Abrir proyecto en el editor de código? (y/n)" open_editor
    if [ $open_editor = "y" ]; then
        code .
        sleep 3
    fi


    echo "Abrir proyecto en el navegador? (y/n)"
    read open_browser
    if [ $open_browser = "y" ]; then
        xdg-open "http://$domain_name.dev.com"
    fi

    exit 1


}

```