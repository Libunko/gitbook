```
obj-app = ad9361_gbcom
obj-dep = ad9361.o

CC = /opt/arm-2010.09/bin/arm-none-linux-gnueabi-gcc
CFLAGS = -Wall -g
LDFLAGS =

.PHONY: all
all: $(obj-app)
$(obj-app): $(obj-dep)
	$(CC) -o $@ $^ $(LDFLAGS)

%.o: %.c
	$(CC) -o $@ -c $< $(CFLAGS)

%.d: %.c
	@set -e; rm -f $@; \
	$(CC) -MM $< $(CFLAGS) > $@.$$$$; \
	sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.$$$$ > $@; \
	rm -f $@.$$$$

-include $(obj-dep:.o=.d)

.PHONY: clean
clean:
	rm -f $(obj-app) *.o *.d *.d.*
```
