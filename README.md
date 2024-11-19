# Download Link

Click the button below to download the file:

[![Download File](https://img.shields.io/badge/Download-ALLPR.txt-blue)](https://shankarhere.blob.core.windows.net/shankarsonar/ALLPR.txt)
[![Download FactorialPrimeApplet.txt](https://img.shields.io/badge/Download-FactorialPrimeApplet.txt-blue)](https://shankarhere.blob.core.windows.net/shankarsonar/FactorialPrimeApplet.txt)
[![Download Aglet.txt](https://img.shields.io/badge/Download-Aglet.txt-blue)](https://shankarhere.blob.core.windows.net/shankarsonar/Aglet.txt)  
[![Download MPI.txt](https://img.shields.io/badge/Download-MPI.txt-blue)](https://shankarhere.blob.core.windows.net/shankarsonar/MPI.txt)  
[![Download MR.txt](https://img.shields.io/badge/Download-MR.txt-blue)](https://shankarhere.blob.core.windows.net/shankarsonar/MR.txt)  
[![Download Multithread application.txt](https://img.shields.io/badge/Download-Multithread%20application.txt-blue)](https://shankarhere.blob.core.windows.net/shankarsonar/Multithread%20application.txt)  
[![Download RMI.txt](https://img.shields.io/badge/Download-RMI.txt-blue)](https://shankarhere.blob.core.windows.net/shankarsonar/RMI.txt)


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




```markdown
# FactorialPrimeApplet

This project contains an applet that allows users to calculate the factorial of a number and check if a number is prime.

## Files

### 1. FactorialPrimeApplet.java

This Java applet provides a user interface with buttons for calculating the factorial of a number and checking if a number is prime.

```java
import java.applet.*;
import java.awt.*;
import java.awt.event.*;

public class FactorialPrimeApplet extends Applet implements ActionListener {
    TextField inputField;
    Button factorialButton, primeButton;
    Label resultLabel;

    public void init() {
        inputField = new TextField(10);
        factorialButton = new Button("Calculate Factorial");
        primeButton = new Button("Check Prime");
        resultLabel = new Label("Result will be displayed here");

        add(new Label("Enter a number:"));
        add(inputField);
        add(factorialButton);
        add(primeButton);
        add(resultLabel);

        // Set action listeners
        factorialButton.addActionListener(this);
        primeButton.addActionListener(this);
    }

    public void actionPerformed(ActionEvent e) {
        try {
            int number = Integer.parseInt(inputField.getText());
            if (e.getSource() == factorialButton) {
                resultLabel.setText("Factorial: " + calculateFactorial(number));
            } else if (e.getSource() == primeButton) {
                resultLabel.setText(number + " is " + (isPrime(number) ? "Prime" : "Not Prime"));
            }
        } catch (NumberFormatException ex) {
            resultLabel.setText("Please enter a valid integer.");
        }
    }

    private int calculateFactorial(int n) {
        if (n == 0 || n == 1) return 1;
        return n * calculateFactorial(n - 1);
    }

    private boolean isPrime(int n) {
        if (n <= 1) return false;
        for (int i = 2; i <= Math.sqrt(n); i++) {
            if (n % i == 0) return false;
        }
        return true;
    }
}
```

### 2. HTML File to run the applet

The HTML file to run the applet in a browser or applet viewer.

```html
<html>
    <body>
        <applet code="FactorialPrimeApplet.class" width="400" height="200"></applet>
    </body>
</html>
```

## How to Run

1. Compile the Java applet:

```bash
javac FactorialPrimeApplet.java
```

2. Run the applet using the applet viewer with the HTML file:

```bash
appletviewer applet.html
```
