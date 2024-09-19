---
layout: single
title:  "Java Primitives"
date:   2024-08-26
categories: Java
---

Demonstration of primitive data types in Java.

{% highlight java %}

public class DefinePrimitives {
    public static void main(String[] args) {
      int anInt = 100;
      double aDouble = 89.9;
      boolean aBoolean = true;

      // not primitives but essential
      String aString = "Hello, World!";   // wrapper class shortcut assignment
      String aStringFormal = new String("Greetings, World!");
  
      System.out.println("anInt: " + anInt);
      System.out.println("aDouble: " + aDouble);
      System.out.println("aBoolean: " + aBoolean);
      System.out.println("aString: " + aString);
      System.out.println("aStringFormal: " + aStringFormal);
    }
  }
{% endhighlight %}

{% highlight java %}
// java style to import library
import java.util.Scanner;

// class must alway have 1st letter as uppercase, CamelCase is Java Class convention
public class ScanPrimitives {
    public static void main(String[] args) {    
        Scanner input;

        // primitive int
        input = new Scanner(System.in);
        System.out.print("Enter an integer: ");
        try {
            int sampleInputInt = input.nextInt();
            System.out.println(sampleInputInt);
        } catch (Exception e) {  // if not an integer
            System.out.println("Not an integer (form like 159), " + e);
        }
        input.close();

        // primitive double
        input = new Scanner(System.in);
        System.out.print("Enter a double: ");
        try {
            double sampleInputDouble = input.nextDouble();
            System.out.println(sampleInputDouble);
        } catch (Exception e) {  // if not a number
            System.out.println("Not an double (form like 9.99), " + e);
        }
        input.close();

        // primitive boolean
        input =  new Scanner(System.in);
        System.out.print("Enter a boolean: ");
        try {
            boolean sampleInputBoolean = input.nextBoolean();
            System.out.println(sampleInputBoolean);
        } catch (Exception e) {  // if not true or false
            System.out.println("Not an boolean (true or false), " + e);
        }
        input.close();

        // wrapper class String
        input =  new Scanner(System.in);
        System.out.print("Enter a String: ");
        try {
            String sampleInputString = input.nextLine();
            System.out.println(sampleInputString);
        } catch (Exception e) { // this may never happen
            System.out.println("Not an String, " + e);
        }
        input.close();
    }
}
{% endhighlight %}

{% highlight java %}
public class PrimitiveDivision {
    public static void main(String[] args) {
        int i1 = 7, i2 = 2;
        System.out.println("Integer Division");
        System.out.println("\tint output with concatenation: " + i1 + "/" + i2 + " = " + i1/i2);
        System.out.println(String.format("\tint output with format: %d/%d = %d",i1, i2, i1/i2));
        System.out.printf("\tint output with printf: %d/%d = %d\n",i1, i2, i1/i2);

        double d1 = 7, d2 = 2;
        System.out.println("Double Division");
        System.out.println("\tdouble output with concatenation: " + d1 + "/" + d2 + " = " + d1/d2);
        System.out.println(String.format("\tdouble output with format: %.2f/%.2f = %.2f",d1, d2, d1/d2));
        System.out.printf("\tdouble output with printf: %.2f/%.2f = %.2f\n",d1, d2, d1/d2);

        System.out.println("Casting and Remainders");
        System.out.printf("\tint cast to double on division: %d/%d = %.2f\n",i1, i2, i1/(double)i2);
        System.out.println("\tint using modulo for remainder: " + i1 + "/" + i2 + " = " + i1/i2 + " remainder " + i1%i2);
    }
}
{% endhighlight %}

{% highlight java %}
public class GradeCalculator {
    // introduction to Double wrapper class (object)
    ArrayList<Double> grades;   // similar to Python list

    // constructor, initializes ArrayList and call enterGrades method
    public GradeCalculator() {
        this.grades = new ArrayList<>();
        this.enterGrades();
    }

    // double requires test for zero versus threshold, DO NOT compare to Zero
    private boolean isZero(double value){
        double threshold = 0.001;
        return value >= -threshold && value <= threshold;
    }


    // enterGrades input method using scanner
    private void enterGrades() {
        Scanner input;

        while (true) {
            input = new Scanner(System.in);
            System.out.print("Enter a double, 0 to exit: ");
            try {
                double sampleInputDouble = input.nextDouble();
                System.out.println(sampleInputDouble);
                if (isZero(sampleInputDouble)) break;       // exit loop on isZero
                else this.grades.add(sampleInputDouble);    // adding to object, ArrayList grades
            } catch (Exception e) {  // if not a number
                System.out.println("Not an double (form like 9.99), " + e);
            }
            input.close();
        }
    }

    // average calculation 
    public double average() {
        double total = 0;   // running total
        for (double num : this.grades) {    // enhanced for loop
            total += num;   // shortcut add and assign operator
        }
        return (total / this.grades.size());  // double math, ArrayList grades object maintains its size
    }

    // static main method, used as driver and tester
    public static void main(String[] args) {
        GradeCalculator grades = new GradeCalculator(); // calls constructor, creates object, which calls enterGrades
        System.out.println("Average: " + String.format("%.2f", grades.average()));  // format used to standardize to two decimal points
    }
}
{% endhighlight %}