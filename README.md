Apologies for the confusion. Here is the entire content of the README file in one single block, without any divisions:

```
# Project: Hadoop, RMI, MPI, Aglets, and Java Networking Examples

This repository contains a collection of various Java-based projects, including Hadoop, RMI, MPI, Aglets, and Java Networking examples. Below is a detailed list of each part of the project, along with the respective Java code.

## 1. Hadoop Setup and Compilation

### Setting up Java and Hadoop

```
ls /usr/lib/jvm

export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64 
export PATH=$JAVA_HOME/bin:$PATH
which java
export HADOOP_HOME=/home/cnlab/Desktop/DC/hadoop-2.10.2
export PATH=$HADOOP_HOME/bin:$PATH
```

### Hadoop Compilation and Execution

```
hadoop com.sun.tools.javac.Main MaxTemperature.java MaxTemperatureMapper.java MaxTemperatureReducer.java
jar cf wc.jar MaxTemperature.class MaxTemperatureMapper.class MaxTemperatureReducer.class
hadoop jar wc.jar MaxTemperature 1901 output
cat output/*
```

## 2. MinTemperature Hadoop Job (PR 7 and 5)

### `MinTemperature.java`

```java
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class MinTemperature {

  public static void main(String[] args) throws Exception {
    if (args.length != 2) {
      System.err.println("Usage: MinTemperature <input path> <output path>");
      System.exit(-1);
    }

    Job job = new Job();
    job.setJarByClass(MinTemperature.class);
    job.setJobName("Min temperature");

    FileInputFormat.addInputPath(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));

    job.setMapperClass(MinTemperatureMapper.class);
    job.setReducerClass(MinTemperatureReducer.class);

    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);

    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}
```

### `MinTemperatureMapper.java`

```java
import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class MinTemperatureMapper extends Mapper<LongWritable, Text, Text, IntWritable> {

  private static final int MISSING = 9999;

  @Override
  public void map(LongWritable key, Text value, Context context)
      throws IOException, InterruptedException {

    String line = value.toString();
    // Assuming the sensor ID is in a specific position, let's say 0 to 10
    String sensorId = line.substring(0, 10).trim();  // Adjust this according to your actual data format
    int airTemperature;

    if (line.charAt(87) == '+') {
      airTemperature = Integer.parseInt(line.substring(88, 92));
    } else {
      airTemperature = Integer.parseInt(line.substring(87, 92));
    }

    String quality = line.substring(92, 93);
    if (airTemperature != MISSING && quality.matches("[01459]")) {
      context.write(new Text(sensorId), new IntWritable(airTemperature));
    }
  }
}
```

### `MinTemperatureReducer.java`

```java
import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class MinTemperatureReducer extends Reducer<Text, IntWritable, Text, IntWritable> {

  @Override
  public void reduce(Text key, Iterable<IntWritable> values, Context context)
      throws IOException, InterruptedException {

    int minValue = Integer.MAX_VALUE;
    for (IntWritable value : values) {
      minValue = Math.min(minValue, value.get());
    }
    context.write(key, new IntWritable(minValue));  // Key is sensor ID, value is min temperature
  }
}
```

## 3. Calculator Multi-Threaded Server (PR 6)

### `CalculatorClient.java`

```java
import java.io.*;
import java.net.*;

public class CalculatorClient {
    private static final String SERVER_ADDRESS = "localhost";
    private static final int SERVER_PORT = 12345;

    public static void main(String[] args) {
        try (Socket socket = new Socket(SERVER_ADDRESS, SERVER_PORT);
             PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
             BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
             BufferedReader consoleInput = new BufferedReader(new InputStreamReader(System.in))) {

            System.out.println("Connected to server. Enter two numbers:");

            String userInput = consoleInput.readLine();
            out.println(userInput);

            String serverResponse = in.readLine();
            System.out.println(serverResponse); 

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### `MultiThreadedServer.java`

```java
import java.io.*;
import java.net.*;

public class MultiThreadedServer {
    private static final int PORT = 12345;

    public static void main(String[] args) {
        try (ServerSocket serverSocket = new ServerSocket(PORT)) {
            System.out.println("Server started on port: " + PORT);
            while (true) {
                Socket clientSocket = serverSocket.accept();
                System.out.println("New client connected: " + clientSocket.getInetAddress().getHostAddress());
                new ClientHandler(clientSocket).start();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

class ClientHandler extends Thread {
    private final Socket socket;
    public ClientHandler(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        try (BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
             PrintWriter out = new PrintWriter(socket.getOutputStream(), true)) {
            String inputLine;
            while ((inputLine = in.readLine()) != null) {
                String response = processCommand(inputLine);
                out.println(response);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    private String processCommand(String command) {
        String[] parts = command.split(" ");
        if (parts.length != 2) {
            return "Invalid command. Use format: [number1] [number2]";
        }

        try {
            double num1 = Double.parseDouble(parts[0]);
            double num2 = Double.parseDouble(parts[1]);

            double sum = num1 + num2;
            double difference = num1 - num2;
            double product = num1 * num2;
            String divisionResult = num2 != 0 ? String.valueOf(num1 / num2) : "Error: Division by zero";
            
            return String.format("Results: Addition: %.2f, Subtraction: %.2f, Multiplication: %.2f, Division: %s",
                                 sum, difference, product, divisionResult);
        } catch (NumberFormatException e) {
            return "Invalid number format.";
        }
    }
}
```

## 4. Aglets (PR 8)

First Extract this zip in Home Dir and go to `/java/aglet/bin`:

```bash
chmod 755 ant
./ant
ant install-home
export AGLETS_HOME=/java/aglet
export AGLETS_PATH=$AGLETS_HOME
export PATH=$PATH:$AGLETS_HOME/bin
```

Run Aglets daemon:

```bash
agletsd -f ../cnf/aglets.props   or  agletsd   or   ./agletsd
```

Use one of the following usernames:
- `anonymous`  
- `aglet_key`

Password:
- `aglets`

### `MyAglet.java`

```java
package myfirst;
import com.ibm.aglet.*;

public class MyAglet extends Aglet {
    public void run() {
        System.out.println("Hello, world!");
        setText("you can see this in the Tahiti window");
    }
}
```

## 5. RMI (PR 9)

### `Calculator.java`

```java
import java.rmi.*;

public interface Calculator extends Remote {
    int add(int x, int y) throws RemoteException;
    int subtract(int x, int y) throws RemoteException;
    int multiply(int x, int y) throws RemoteException;
    int divide(int x, int y) throws RemoteException;
}
```

### `CalculatorImpl.java`

```java
import java.rmi.*;
import java.rmi.server.*;

public class CalculatorImpl extends UnicastRemoteObject implements Calculator {
    public CalculatorImpl() throws RemoteException {
        super();
    }

    public int add(int x, int y) {
        return x + y;
    }

    public int subtract(int x, int y) {
        return x - y;
    }

    public int multiply(int x, int y) {
        return x * y;
    }

    public int divide(int x, int y) {
        if (y != 0) {
            return x / y;
        } else {
            return -1; // return -1 if division by zero
        }
    }
}
```

### `Server.java`

```java
import java.rmi.*;
import java.rmi.registry.*;

public class Server {
    public static void main

(String[] args) {
        try {
            CalculatorImpl calc = new CalculatorImpl();
            Registry registry = LocateRegistry.createRegistry(1099);
            registry.rebind("Calculator", calc);
            System.out.println("Calculator service is running...");
        } catch (Exception e) {
            System.out.println("Server exception: " + e.toString());
            e.printStackTrace();
        }
    }
}
```

### `Client.java`

```java
import java.rmi.*;

public class Client {
    public static void main(String[] args) {
        try {
            Calculator calc = (Calculator) Naming.lookup("//localhost/Calculator");
            int x = 10, y = 5;
            System.out.println("Addition: " + calc.add(x, y));
            System.out.println("Subtraction: " + calc.subtract(x, y));
            System.out.println("Multiplication: " + calc.multiply(x, y));
            System.out.println("Division: " + calc.divide(x, y));
        } catch (Exception e) {
            System.out.println("Client exception: " + e.toString());
            e.printStackTrace();
        }
    }
}
```

---

This is the full content of the README file you requested. You can copy and paste this entire block at once.
