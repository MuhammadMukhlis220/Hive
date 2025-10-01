## KERBEROS HIVE IN DBEAVER - Windows

Source doc: https://docs.arenadata.io/en/ADH/current/tutorials/security/kerberos/hive-dbeaver.html

Requirement
1. Windows
2. Dbeaver Community Edition 25.0.0 (I only tested in that version)

Intro: CLOSE YOUR DBEAVER

### 1. Get krb5.conf file from your linux server 

Default in /etc/krb5.conf and put it in your dbeaver installation in DBeaver\jre\conf\security

location in /etc/krb5.conf

Mine directory: C:\Users\mmukh\AppData\Local\DBeaver\jre\conf\security\krb5.conf

### 2. Edit krb5.conf file

Clear all symbol or anything before [libdefaults] and make renew_lifetime get commented

krb5.conf example:

[libdefaults]
  #renew_lifetime = 7d
  forwardable = true
  default_realm = MENINJOCLOUD.COM
  ticket_lifetime = 24h
  dns_lookup_realm = false
  dns_lookup_kdc = false
  rdns = false

[domain_realm]
  .meninjocloud.com = MENINJOCLOUD.COM
  meninjocloud.com = MENINJOCLOUD.COM

[logging]
  default = FILE:/var/log/krb5kdc.log
  admin_server = FILE:/var/log/kadmind.log
  kdc = FILE:/var/log/krb5kdc.log

[realms]
  MENINJOCLOUD.COM = {
    admin_server = ad.meninjocloud.com
    kdc = ad.meninjocloud.com:88
  }

### 3. Get hive.keytab

In my case: file name is hive.service.keytab. Location from 10.10.6.41 in `/etc/security/keytabs/hive.security.keytab`
Place it in your local

Mine directory: C:\Users\mmukh\Documents\Mukhlis\hive.service.keytab

### 4. Create the jaas.conf file

Put it inside the DBeaver folder. 

In this file, you should specify the path to the principal keytab file used for connecting to Hive. 
You also have to specify the principal name.

To get principal name:
klist -kt <path to your hive keytab>

it will show the available principal

jaas.conf example:

Client {
  com.sun.security.auth.module.Krb5LoginModule required
  doNotPrompt=true
  useKeyTab=true
  keyTab="C:/Users/mmukh/Documents/Mukhlis/hive.service.keytab"
  useTicketCache=false
  renewTGT=false
  principal="hive/stg-onyxmns02.meninjocloud.id@MENINJOCLOUD.COM";
};

Mine dir: C:\Users\mmukh\Documents\Mukhlis\jaas.conf

### 5. Edit dbeaver.ini

In my case, dbeaver.ini already in my dbeaver folder: C:\Users\mmukh\AppData\Local\DBeaver\dbeaver.ini

Add some java command bellow in last line:

-Djava.security.krb5.conf=C:\Users\mmukh\AppData\Local\DBeaver\jre\conf\security\krb5.conf
-Djava.security.auth.login.config=C:\Users\mmukh\Documents\Mukhlis\jaas.conf
-Djavax.security.auth.useSubjectCredsOnly=false

dbeaver.ini example:

-startup
plugins/org.jkiss.dbeaver.launcher_1.0.24.202503021833.jar
--launcher.library
plugins/org.eclipse.equinox.launcher.win32.win32.x86_64_1.2.1100.v20240722-2106
-vmargs
-XX:+IgnoreUnrecognizedVMOptions
-Dosgi.requiredJavaVersion=17
-Dfile.encoding=UTF-8
--add-modules=ALL-SYSTEM
--add-opens=java.base/java.io=ALL-UNNAMED
--add-opens=java.base/java.lang=ALL-UNNAMED
--add-opens=java.base/java.lang.reflect=ALL-UNNAMED
--add-opens=java.base/java.net=ALL-UNNAMED
--add-opens=java.base/java.nio=ALL-UNNAMED
--add-opens=java.base/java.nio.charset=ALL-UNNAMED
--add-opens=java.base/java.text=ALL-UNNAMED
--add-opens=java.base/java.time=ALL-UNNAMED
--add-opens=java.base/java.util=ALL-UNNAMED
--add-opens=java.base/java.util.concurrent=ALL-UNNAMED
--add-opens=java.base/java.util.concurrent.atomic=ALL-UNNAMED
--add-opens=java.base/jdk.internal.vm=ALL-UNNAMED
--add-opens=java.base/jdk.internal.misc=ALL-UNNAMED
--add-opens=java.base/sun.nio.ch=ALL-UNNAMED
--add-opens=java.base/sun.nio.fs=ALL-UNNAMED
--add-opens=java.base/sun.security.ssl=ALL-UNNAMED
--add-opens=java.base/sun.security.action=ALL-UNNAMED
--add-opens=java.base/sun.security.util=ALL-UNNAMED
--add-opens=java.security.jgss/sun.security.jgss=ALL-UNNAMED
--add-opens=java.security.jgss/sun.security.krb5=ALL-UNNAMED
--add-opens=java.desktop/java.awt=ALL-UNNAMED
--add-opens=java.desktop/java.awt.peer=ALL-UNNAMED
--add-opens=java.sql/java.sql=ALL-UNNAMED
-Xms64m
-Xmx1024m
-Ddbeaver.distribution.type=exe


-Djava.security.krb5.conf=C:\Users\mmukh\AppData\Local\DBeaver\jre\conf\security\krb5.conf
-Djava.security.auth.login.config=C:\Users\mmukh\Documents\Mukhlis\jaas.conf
-Djavax.security.auth.useSubjectCredsOnly=false


### 6. Edit known windows hosts 

Edit hosts in: C:\Windows\System32\drivers\etc\hosts

add:

<ip> <kdc value in krb5.conf file (realms section)>

Mine:

10.10.5.155 ad.meninjocloud.com

Get the ip by enter your server where krb5.conf you get and run: ping ad.meninjocloud.com

It will show the ip of your kdc active server. Write the ip number in hosts.

### 7. Run bash command

Go to bin folder using powershell, mine in: C:\Users\mmukh\AppData\Local\DBeaver\jre\bin
and run: .\kinit.exe -k -t <dir-to-your-keytab-file> <hive principal>

Mine example:
bash
cd C:\Users\mmukh\AppData\Local\DBeaver\jre\bin
.\kinit.exe -k -t "C:\Users\mmukh\Documents\Mukhlis\hive.service.keytab" hive/stg-onyxmns02.meninjocloud.id@MENINJOCLOUD.COM

### 8. Download driver jar

https://repo1.maven.org/maven2/io/arenadata/hive/hive-jdbc/3.1.1-arenadata/hive-jdbc-3.1.1-arenadata-standalone.jar

### 9. Now run the dbeaver

### 10. Edit Driver

Go to Database -> Driver Manager -> New

Fill:
Driver Name: Hive Kerberos
URL Template: jdbc:hive2://{host}:{port}/{database};principal=hive/stg-onyxmns02.meninjocloud.id@MENINJOCLOUD.COM;
Default Port: 10000
Default Database: Default

Go to Libraries -> Add File (Select your downloaded jar)
Find Class -> Select org.apache.hive.jdbc.HiveDriver

Click Ok

### 11. Create New Connection to Hive

Create new connection and select "Hive Kerberos". Fill username, password, port, and database. It should connect without error.

### Limitation:

krb5.conf only provide one realm of Kerberos. So our dbeaver can only launch the start up to only one realm. If there are another realm we want to add, the only option is duplicate the dbeaver folder. So there will be more than one dbeaver with different start up.

