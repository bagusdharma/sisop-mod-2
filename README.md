# Modul 2
Proses dan Daemon

## 1 Process

### 1.1 Process
TL; DR.

Program yang sedang berjalan disebut dengan proses. 

Jika kamu mempunyai dua program yang tampil pada monitor, berarti terdapat dua proses yang sedang berjalan. Proses yang dijalankan pada terminal (shell) akan menghentikan proses shell, kemudian setelah proses selesai, shell akan dijalankan kembali.

Jalankan:
```
$ xlogo
```

### 1.2 Process ID
Setiap proses memiliki dikenali dengan _Process ID_/_pid_. Setiap proses memiliki _Parent Process ID_/_ppid_, kecuali proses `init` atau `systemd`. 

Jalankan: 
```
$ pstree
```
```
init-+-acpid
     |-apache2---7*[apache2]
     |-atd
     |-cron
     |-dbus-daemon
     |-dockerd-+-docker-containe---10*[{docker-containe}]
     |         `-10*[{dockerd}]
     |-fail2ban-server---2*[{fail2ban-server}]
     |-freshclam
```

__Explanation__
  - [pstree](https://linux.die.net/man/1/pstree): ...

Contoh program `demo-process.c`
```C
#include <stdio.h>
#include <unistd.h>

int main() {
  printf("The process ID is %d\n", (int) getpid());
  printf("The parent process ID is %d\n", (int) getppid());
  return 0;
}
```
Hasil
```
The process ID is 8295
The parent process ID is 29043
```
__Explanation__
  - getpid(): ...
  - getppid(): ...


### 1.3 Melihat Proses
Jalankan program

```bash
$ ps -e -o pid,ppid,command
```
```
  PID  PPID COMMAND
    1     0 /sbin/init splash
    2     0 [kthreadd]
    6     2 [ksoftirqd/0]
    7     2 [rcu_sched]
    8     2 [rcu_bh]
    9     2 [migration/0]
   10     2 [lru-add-drain]
   11     2 [watchdog/0]
 .......
 (long list)
 .......
 3760  2684 /usr/lib/x86_64-linux-gnu/notify-osd
 3789  2684 /opt/google/chrome/chrome
25793  9789 ps -e -o pid,ppid,command
```

__Explanation__
  - ps: ...
  - -e: ...
  - -o: ...


### 1.4 Membunuh Proses
Membunuh proses menggunakan `$ kill {pid}`

Contoh: 
```
$ kill 3789
```

### 1.5 Membuat Proses
Proses dapat dibuat menggunakan dua cara (pada C), yaitu dengan `system()` atau `fork` & `exec`

#### 1.5.1 Menggunakan `system()`
   
Ketika `system()` dijalankan, ia akan memanggil standard shell (`/bin/bash`) dan menjalankan perintah yang diminta.

Contoh

```C
#include <stdlib.h>

int main() {
  int return_value;
  return_value = system("ls -l /");
  return return_value;
}
```
   
Hasil
   
```
total 156
drwxr-xr-x   2 root root  4096 Sep 14 06:35 bin
drwxr-xr-x   4 root root  4096 Sep 20 00:24 boot
drwxrwxr-x   2 root root  4096 Agu 14 14:05 cdrom
drwxr-xr-x   3 root root  4096 Sep 12 19:11 data
(long list)
```

#### 1.5.2 Menggunakan `fork` dan `exec`
   
TL;DR.  
`fork` digunakan untuk menduplikasi program yang sedang berjalan.  
`exec` digunakan untuk mengganti program yang sedang berjalan dengan program yang baru.  

#### A. `fork` explained

Ketika `fork` dijalankan, proses baru yang disebut _child process_ akan dibuat. _Parent process_ tetap berjalan dan _child process_ mulai dibuat dan berjalan ketika function `fork` dipanggil.

```C
int main() { 
                            pid: 23, ppid: 10 
                             [Main process]
                                 |
  fork();              > Child process created <
                                 +
                               /   \
                             /       \
               pid: 23, ppid: 10    pid: 30, ppid: 23
                [Parent Process]    [Child Process]

  return 0;
}
```

Real code:
```C
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
  pid_t child_pid;

  printf("The main program PID is %d\n\n", (int) getpid());

  child_pid = fork();
  if (child_pid != 0) {
    printf("This is the parent process, with PID %d\n", (int) getpid());
    printf("The child's PID is %d\n", (int) child_pid);
  } else {
    printf("This is the child process, with PID %d\n", (int) getpid());
  }

  return 0;
}
```

__Explanation__
  - ....

Lengkapi guys

#### B. `exec` explained


Diisi ya penjelasannya  
Halaman 6 http://advancedlinuxprogramming.com/alp-folder/alp-ch03-processes.pdf

Example: [exec-sample.c](https://github.com/syukronrm/sisop-mod-2/blob/master/sample-exec.c)

#### C. `fork` and `exec` explained!
__Permasalahan:__
Bagaimana cara menjalankan dua proses dalam satu program?

__Contoh Permasalahan:__
Bagaimana cara membuat folder `~/sisop` dan membuat file kosong bernama `~/process.c`?

Maka, bagaimana cara menjalankan `mkdir` __dan__ `touch` dalam satu program?

__Solusi:__
Gunakan `fork` dan `exec`!

TL;DR.  
Buat sebuah program dengan:  
1. Buat proses baru dengan `fork`
2. Jalankan `exec` yang memanggil `mkdir` pada child process
3. Jalankan `exec` yang memanggil `touch` pada parent process

Visualisasi
```
+--------+
| pid=7  |
| ppid=4 |
| bash   |
+--------+
    |
    | calls fork
    V                         
+--------+                     +--------+
| pid=7  |    forks            | pid=22 |
| ppid=4 | ------------------> | ppid=7 |
| bash   |                     | bash   |
+--------+                     +--------+
    |                              |
    | calls exec to run touch      | calls exec to run mkdir
    |                              |
    V                              V
```

Example: [sample-fork-exec.c](https://github.com/syukronrm/sisop-mod-2/blob/master/sample-fork-exec.c)

#### D. `wait` explained!

`wait` adalah function yang digunakan untuk memblock program yang sedang berjalan hingga proses child processnya berhenti.

__Permasalahan:__  
Bagaimana cara membuat program yang menjalankan suatu proses tanpa menghentikan program?

__Contoh permasalahan:__  
Bagaimana cara membuat folder `~/sisop` dan membuat file kosong bernama `~/process.c` __di dalamnya__?

Maka, bagaimana cara menjalankan `mkdir` __lalu__ menjalankan `touch` dalam satu program?

__Solusi:__  
Gunakan `fork`, `exec`, dan `wait`!

Buat sebuah program dengan:  
1. Buat proses baru dengan `fork`
2. Jalankan `exec` yang memanggil `mkdir` pada child process
3. Buat parent process menunggu (`wait`) hingga proses pada child selesai
4. Setelah child selesai, jalankan `exec` yang memanggil `touch` pada parent

Visualisasi
```
+--------+
| pid=7  |
| ppid=4 |
| bash   |
+--------+
    |
    | calls fork
    V
+--------+             +--------+
| pid=7  |    forks    | pid=22 |
| ppid=4 | ----------> | ppid=7 |
| bash   |             | bash   |
+--------+             +--------+
    |                      |
    | waits for pid 22     | calls exec to run mkdir
    |                      V
    |                  +--------+
    |                  | pid=22 |
    |                  | ppid=7 |
    |                  | ls     |
    V                  +--------+
+--------+                 |
| pid=7  |                 | exits
| ppid=4 | <---------------+
| bash   |
+--------+
    |
    | calls exec to run touch
    |
    V
```

Example: [sample-fork-exec-wait.c](https://github.com/syukronrm/sisop-mod-2/blob/master/sample-fork-exec-wait.c)

## 1.6 Jenis-Jenis Proses
### 1.6.1 Zombie Process
Jika child process telah berhenti dan parent process memanggil function `wait`, maka child process akan hilang dan exit status dari child akan didapat dari pemanggilan function `wait`. Apa yang terjadi jika child process berhenti dan parent tidak memanggil `wait`? Apakah child process tersebut tetap hilang? Tidak, child process tersebut akan menjadi zombie process.

```
PID     PPID    STAT  COMMAND
28621   31403   S+    ./sample-zombie-process
28622   28621   Z+    [sample-zombie-p] <defunct>
```

Status dari zombie process adalah `Z` atau terdapat `<defunct>` di akhir commandnya.

Example: [sample-zombie-process.c](https://github.com/syukronrm/sisop-mod-2/blob/master/sample-zombie-process.c)

### 1.6.2 Orphan Process
Orphan process adalah child process yang yang parent processnya telah berhenti.

```
PID    PPID     STAT  COMMAND
    1     0     Ss    /sbin/init
28369     1     S     ./sample-orphan-process
```

Example: [sample-orphan-process.c](https://github.com/syukronrm/sisop-mod-2/blob/master/sample-orphan-process.c)

### 1.6.3 Daemon Process
Daemon process adalah proses yang berjalan di balik layar (background) dan tidak dapat berinteraksi dengan user melalui standard input/output. Akan dijelaskan di bab selanjutnya.

## 2. Daemon
### 2.1 Daemon
Daemon process adalah proses yang berjalan di balik layar (background) dan tidak dapat berinteraksi dengan user melalui standard input/output.

### 2.2 Membuat Daemon


# Appendix
### Useful things
#### Get all libraries documentation (and functions)
```
# apt-get install manpages-posix-dev

$ man {anything-you-want-to-know}
$ man fopen
$ man fclose
$ man unistd.h
```

### References
https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet  
https://notes.shichao.io/apue/  
http://advancedlinuxprogramming.com/alp-folder/alp-ch03-processes.pdf  
http://www.linuxzasve.com/preuzimanje/TLCL-09.12.pdf  
https://stackoverflow.com/questions/1653340/differences-between-fork-and-exec
