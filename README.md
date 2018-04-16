# How it worked for me
I've found no documentation how to use the entropia libsocket-can-java JNI-Library, so my solution (running with MCP 2515 module) written down:

Get SocketCan running on the Pi, so it shows up "can0" interface in `ifconfig`. You will find a tutorial for this. If it is running, do the following:

1. Clone the libsocket-can-java Library to the Raspberry Pi 3 with `git clone` (There is a Makefile in the cloned directory. It contains all info to compile the C Library as an .so "Shared Library")
2. Compile it with commandline `make` (This requires at least a C compiler like gcc)
3. In the folder the file `lib/libjni_socketcan.so` was created. Copy this shared library file to `/usr/lib`. Linux and the Java programm will find it there now.
4. Create a Java program to test it.

In Java you import the `CanSocket.java` class to your project. An example how you can use it:

## Java Example
```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Start...");

        try {
            // Init
            CanSocket socket = new CanSocket(CanSocket.Mode.RAW);
            CanInterface canInterface = new CanInterface(socket, "can0"); // Must be the name of the can interface you find in ifconfig
            socket.bind(canInterface);

            // Reading
            for (int i = 0; i < 5; i++) {
                System.out.println(Arrays.toString(socket.recv().getData())); // Will hang on if there is no data to receive.
                sleep(100);
            }

            {
                // Sending Standard Format
                byte[] data = {0x02};
                int address = 0x1F;
                CanFrame frame = new CanFrame(canInterface, new CanSocket.CanId(address), data);
                System.out.println("Sending frame... Addr: " + address + " Data: " + Arrays.toString(data));
                socket.send(frame);
            }

            {
                // Sending Extended Format - notice setEFFSFF()
                byte[] data = {0x01};
                int address = 0x56CCCCC;            
                CanFrame frame = new CanFrame(canInterface, new CanSocket.CanId(address).setEFFSFF(), data);
                System.out.println("Sending frame... Addr: " + address + " Data: " + Arrays.toString(data));
                socket.send(frame);
            }

            // Close the socket. Maybe optional
            socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static void sleep(int ms){
        try {
            Thread.sleep(ms);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
## Notice
I can't serve any help and won't improve it, I am not working on it - I only use it and wanted to share my way how to get it work.

*Keywords: Socket CAN Java, JNI, Linux, Raspberry Pi, Raspbian, ARM*
