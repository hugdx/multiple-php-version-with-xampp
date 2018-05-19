### Reason 
I and a lot developer work with multiple project at same time, sometimes we are need working with multiple php version 
at same time. We don't want to run multiple XAMPP with different ports. So i created this project to help anybody need
to run multiple php version at same times with only 1 XAMPP

### How it work?
XAMPP use `mod_php` to run php, that mean you only work with 1 php version with xampp. To resolve this, i use `mod_fcgi` 
instead `mod_php`. With `mod_fcgi` we can redirect request to correctly php version we want.

### Install on window
1. Download XAMPP. this guide use [XAMPP 7.2.5 x86](https://sourceforge.net/projects/xampp/files/XAMPP%20Windows/7.2.5/xampp-win32-7.2.5-0-VC15-installer.exe/download)
2. Download [mod_fcgi](https://www.apachelounge.com/download/) . `XAMPP 7.2.5 x86` use `Apache/2.4.33 (Win32)`, so i will download [mod_fcgid-2.3.9-win32-VC15](https://www.apachelounge.com/download/VC15/modules/mod_fcgid-2.3.9-win32-VC15.zip)
3. Install XAMPP. for example i installed to `C:/xampp`. Extract `mod_fcgid-2.3.9-win32-VC15.zip` and copy file `mod_fcgid.so` to `C:/xampp/apache/modules`  
4. Download php version you want from[ https://windows.php.net/downloads/releases/archives/](https://windows.php.net/downloads/releases/archives/). Unzip this to folder `C:/xampp`. for example i will download [php-5.6.35-nts-Win32-VC11-x86](https://windows.php.net/downloads/releases/archives/php-5.6.35-nts-Win32-VC11-x86.zip) and unzip to `C:/xampp/php5635`. rename `php5635/php.ini-development` to `php5635/php.ini` and add this line to end of `php.ini`
   ```
    [custom_php_variable]
    extension_dir       = "C:/xampp/php5635/ext/"
    session.save_path   = "C:/xampp/tmp/"
    upload_tmp_dir      = "C:/xampp/tmp/"
    sys_temp_dir        = "C:/xampp/tmp/"
    soap.wsdl_cache_dir = "C:/xampp/tmp/"
   ```
5. Copy directory [apache](https://github.com/hugdx/multiple-php-version-with-xampp/apache) to `C:/xampp/apache` and edit file `C:/xampp/apache/conf/php.d/vars.conf`
    ```
    # Change this to your xampp on your computer. for this example, it is C:/xampp
    Define XAMPP_DIR        "C:/xampp"
    
    # In step #4, i downloaded php 5.6.35, i will add path to here
    Define PHP_56_CGI       "${XAMPP_DIR}/php5635/php-cgi.exe"
    Define PHP_56_RC        "${XAMPP_DIR}/php5635/"
    
    # If you download more php versions, just add more lines here, for example php 5.6.30
    # Define PHP_5630_CGI   "${XAMPP_DIR}/php5630/php-cgi.exe"
    # Define PHP_5630_RC    "${XAMPP_DIR}/php5630/"
    
    # The default php version of XAMPP 7.2.5 x86 is php 7.2.5
    Define PHP_72_CGI       "${XAMPP_DIR}/php/php-cgi.exe"
    Define PHP_72_RC        "${XAMPP_DIR}/php"
    
    # This will set default php version when run with xampp
    Define PHP_CGI          ${PHP_72_CGI}
    Define PHP_RC           ${PHP_72_RC}
    ```
6. Edit file `C:/xampp/apache/conf/extra/httpd-xampp.conf`. 
    Comment lines and add more 1 line `Include "conf/php.d/multi-php-versions.conf"`
    ```
    #
    # PHP-Module setup
    #
    #LoadFile "C:/xampp/php/php7ts.dll"
    #LoadFile "C:/xampp/php/libpq.dll"
    #LoadModule php7_module "C:/xampp/php/php7apache2_4.dll"
    
    #<FilesMatch "\.php$">
    #    SetHandler application/x-httpd-php
    #</FilesMatch>
    #<FilesMatch "\.phps$">
    #    SetHandler application/x-httpd-php-source
    #</FilesMatch>

    #
    # PHP-CGI setup
    #
    #<FilesMatch "\.php$">
    #    SetHandler application/x-httpd-php-cgi
    #</FilesMatch>
    #<IfModule actions_module>
    #    Action application/x-httpd-php-cgi "/php-cgi/php-cgi.exe"
    #</IfModule>
    
    # Load php multiple versions config 
    Include "conf/php.d/multi-php-versions.conf"
    ```

7. Create your project with vhost: for example i will create project `php-multiple-version` with `php 5.6` and `php 7.2`
    Create vhost file `C:/xampp/apache/conf/vhost/php72-php-multiple-version.local.conf`
    ```
    <VirtualHost *:80>
        ServerAdmin webmaster@admin.local
        DocumentRoot "C:/Projects/php-multiple-version/"
        ServerName php72-php-multiple-version.local
        ErrorLog "logs/php72-php-multiple-version.local-error.log"
        CustomLog "logs/php72-php-multiple-version.local.log" common
    	
    	FcgidInitialEnv PHPRC ${PHP_72_RC}
    	<Directory "C:/Projects/php-multiple-version/">
    		Define PHP_CGI 			${PHP_71_CGI}
    		Include "conf/php.d/php_cgi.conf"
    	</Directory>
    </VirtualHost>
    ```
    
    Create vhost file `C:/xampp/apache/conf/vhost/php56-php-multiple-version.local.conf`
    ```
    <VirtualHost *:80>
        ServerAdmin webmaster@admin.local
        DocumentRoot "C:/Projects/php-multiple-version/"
        ServerName php56-php-multiple-version.local
        ErrorLog "logs/php56-php-multiple-version.local-error.log"
        CustomLog "logs/php56-php-multiple-version.local.log" common
    	
    	FcgidInitialEnv PHPRC ${PHP_56_RC}
    	<Directory "C:/Projects/php-multiple-version/">
    		Define PHP_CGI 			${PHP_56_CGI}
    		Include "conf/php.d/php_cgi.conf"
    	</Directory>
    </VirtualHost>
    ```
    
    add to end of file `C:\Windows\System32\drivers\etc\hosts`
    ```
    	127.0.0.1       php72-php-multiple-version.local
    	127.0.0.1       php56-php-multiple-version.local
    ```
    
    create file `C:/Projects/php-multiple-version/index.php`
    ```php
    <?php phpinfo();
    ```
    
    Start xampp and open your browser: http://php72-php-multiple-version.local and http://php56-php-multiple-version.local
    
### NOTE
1. When stop apache from XAMPP control panel, php-cgi still running in background. to stop it, run this command via cmd
   
    ```
    taskkill /F /IM php-cgi.exe /T
    
    ``` 
2. If you want to run php by command, you can create file .bat like that: `php72.bat`
    ```bat
    C:/xampp/php-7.2.0/php.exe %*    
    ```
    
    and run to test:
    ```bat
    php72 -r "phpinfo();"
    ```