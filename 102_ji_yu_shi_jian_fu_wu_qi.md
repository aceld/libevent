# 10.2 基于事件服务器
##服务端
```cpp
#include <event2/event-config.h>

#include <sys/types.h>
#include <sys/stat.h>
#include <sys/queue.h>
#include <unistd.h>
#include <sys/time.h>
#include <fcntl.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <errno.h>

#include <event.h>

static void
fifo_read(evutil_socket_t fd, short event, void *arg)
{
        char buf[255];
        int len;
        struct event *ev = arg;

        /* Reschedule this event */
        event_add(ev, NULL);

        fprintf(stderr, "fifo_read called with fd: %d, event: %d, arg: %p\n",
            (int)fd, event, arg);
        len = read(fd, buf, sizeof(buf) - 1);

        if (len == -1) {
                perror("read");
                return;
        } else if (len == 0) {
                fprintf(stderr, "Connection closed\n");
                return;
        }

        buf[len] = '\0';

        fprintf(stdout, "Read: %s\n", buf);
}

int
main(int argc, char **argv)
{
        struct event evfifo;

        struct stat st;
        const char *fifo = "event.fifo";
        int socket;

        if (lstat(fifo, &st) == 0) {
                if ((st.st_mode & S_IFMT) == S_IFREG) {
                        errno = EEXIST;
                        perror("lstat");
                        exit(1);
                }
        }

        unlink(fifo);
        if (mkfifo(fifo, 0600) == -1) {
                perror("mkfifo");
                exit(1);
        }

        /* Linux pipes are broken, we need O_RDWR instead of O_RDONLY */
        socket = open(fifo, O_RDONLY | O_NONBLOCK, 0);

        if (socket == -1) {
                perror("open");
                exit(1);
        }

        fprintf(stderr, "Write data to %s\n", fifo);

        /* Initalize the event library */
        event_init();

        /* Initalize one event */
        event_set(&evfifo, socket, EV_READ, fifo_read, &evfifo);

        /* Add it to the active events, without a timeout */
        event_add(&evfifo, NULL);

        event_dispatch();

        return (0);
}

```

##客户端

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <pthread.h>

int main(int argc, char *argv[])
{
    int fd = 0;
    char *str = "hello libevent!";

    fd = open("event.fifo", O_RDWR);
    if (fd < 0) {
        perror("open error");
        exit(1);
    }

    while (1) {
        write(fd, str, strlen(str));
        sleep(1);
    }

    close(fd);

        return 0;
}
```