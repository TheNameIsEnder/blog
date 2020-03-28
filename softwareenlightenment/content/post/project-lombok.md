+++

author = "Jacob Hell"
title = "Java Programming is 90% Easier with Project Lombok"
date = "2020-03-27"
description = "How to use Project Lombok"
tags = [
    "java",
    "project lombok"
]

+++

Writing in Java is a pain in the butt. Even simple code is verbose. Project Lombok condenses Java syntax. This reduction can be up to 90% of normal java code. 

Let's see how Lombok does this.

<!--more-->

## Setting Up

I'm using maven. [Here](https://projectlombok.org/setup/maven) are the Project Lombok maven docs.

I added this dependency:

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.12</version>
    <scope>provided</scope>
</dependency>
```

My IDE is Eclipse.

[Here](https://projectlombok.org/setup/eclipse) are the Eclipse docs.

## Cleaning Up Accessor Methods

My IDE has generated thousands of getters and setters for me. However, going through Eclipse's UI to generate them is a struggle. Good thing that Lombok completely removes the need to do this!

To generate accessor methods for the whole class:

```java
@Getter
@Setter
public class GetterSetterExample {
  private int age = 10;
  private String name;
}
```



Customizing visibility:

```java
public class GetterSetterExample {
  @Getter @Setter private int age = 10;
  
  @Setter(AccessLevel.PROTECTED) private String name;
  
  @Override public String toString() {
    return String.format("%s (age: %d)", name, age);
  }
}
```

## Null Checking

Every time I see `NullPointerException` with unhelpful information, my blood pressure spikes.  We can use the `@NonNull` attribute to make things easier.

```java
public int getStringLength(@NonNull String str) {
    return str.length();
}
```

If I do this:

```java
getStringLength(null);
```

I get this:

```
Exception in thread "main" java.lang.NullPointerException: str is marked non-null but is null
```

## toString

Another bane of a Java Developer's existence: toString() methods. Like the accessor methods, just an annotation is necessary.

```java
@ToString(includeFieldNames = true)
public class Square {
    private final int width, height;

    public Square(int width, int height) {
        this.width = width;
        this.height = height;
    }
}
```

Doing this:

```java
Square square = new Square(5, 10);
System.out.println(square.toString());
```

Will print out:

```
Square(width=5, height=10)
```

## Constructors

There can't be more right? Well, there is. Even constructors aren't immune to the power of Lombok.

Creating a constructor for all variables:

```java
@AllArgsConstructor
public class Square {
   private int width, height;
}
```

No argument constructor:

```java
@NoArgsConstructor
public class Square {
   private int width, height;
}
```

`@RequiredArgsConstructor` will create a constructor for variables that are `final` or have `@NonNull`

```java
@RequiredArgsConstructor
public class Square {
   private int width, height;
}
```



## @Data, the Mother of All Annotations

Still too much code for you? Well, `@Data` combines `@ToString`, `@Getter`, `@Setter`, and `@RequiredArgsConstructor`!

So this evil piece of Java code (117 lines):

```java
public class DataExample {
  private final String name;
  private int age;
  private double score;
  private String[] tags;
  
  public DataExample(String name) {
    this.name = name;
  }
  
  public String getName() {
    return this.name;
  }
  
  void setAge(int age) {
    this.age = age;
  }
  
  public int getAge() {
    return this.age;
  }
  
  public void setScore(double score) {
    this.score = score;
  }
  
  public double getScore() {
    return this.score;
  }
  
  public String[] getTags() {
    return this.tags;
  }
  
  public void setTags(String[] tags) {
    this.tags = tags;
  }
  
  @Override public String toString() {
    return "DataExample(" + this.getName() + ", " + this.getAge() + ", " + this.getScore() + ", " + Arrays.deepToString(this.getTags()) + ")";
  }
  
  protected boolean canEqual(Object other) {
    return other instanceof DataExample;
  }
  
  @Override public boolean equals(Object o) {
    if (o == this) return true;
    if (!(o instanceof DataExample)) return false;
    DataExample other = (DataExample) o;
    if (!other.canEqual((Object)this)) return false;
    if (this.getName() == null ? other.getName() != null : !this.getName().equals(other.getName())) return false;
    if (this.getAge() != other.getAge()) return false;
    if (Double.compare(this.getScore(), other.getScore()) != 0) return false;
    if (!Arrays.deepEquals(this.getTags(), other.getTags())) return false;
    return true;
  }
  
  @Override public int hashCode() {
    final int PRIME = 59;
    int result = 1;
    final long temp1 = Double.doubleToLongBits(this.getScore());
    result = (result*PRIME) + (this.getName() == null ? 43 : this.getName().hashCode());
    result = (result*PRIME) + this.getAge();
    result = (result*PRIME) + (int)(temp1 ^ (temp1 >>> 32));
    result = (result*PRIME) + Arrays.deepHashCode(this.getTags());
    return result;
  }
  
  public static class Exercise<T> {
    private final String name;
    private final T value;
    
    private Exercise(String name, T value) {
      this.name = name;
      this.value = value;
    }
    
    public static <T> Exercise<T> of(String name, T value) {
      return new Exercise<T>(name, value);
    }
    
    public String getName() {
      return this.name;
    }
    
    public T getValue() {
      return this.value;
    }
    
    @Override public String toString() {
      return "Exercise(name=" + this.getName() + ", value=" + this.getValue() + ")";
    }
    
    protected boolean canEqual(Object other) {
      return other instanceof Exercise;
    }
    
    @Override public boolean equals(Object o) {
      if (o == this) return true;
      if (!(o instanceof Exercise)) return false;
      Exercise<?> other = (Exercise<?>) o;
      if (!other.canEqual((Object)this)) return false;
      if (this.getName() == null ? other.getValue() != null : !this.getName().equals(other.getName())) return false;
      if (this.getValue() == null ? other.getValue() != null : !this.getValue().equals(other.getValue())) return false;
      return true;
    }
    
    @Override public int hashCode() {
      final int PRIME = 59;
      int result = 1;
      result = (result*PRIME) + (this.getName() == null ? 43 : this.getName().hashCode());
      result = (result*PRIME) + (this.getValue() == null ? 43 : this.getValue().hashCode());
      return result;
    }
  }
}
```

Turns into this friendly snippet (13 lines):

```java
@Data public class DataExample {
  private final String name;
  @Setter(AccessLevel.PACKAGE) private int age;
  private double score;
  private String[] tags;
  
  @ToString(includeFieldNames=true)
  @Data(staticConstructor="of")
  public static class Exercise<T> {
    private final String name;
    private final T value;
  }
}
```

## Conclusion

[Project Lombok](https://projectlombok.org/) is my best find this year. But wait, there's more!

The [documentation](https://projectlombok.org/features/all) is fantastic and has further features to sink your teeth into.

