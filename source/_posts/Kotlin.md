---
layout: objects
title: Objects in Kotlin:Create safe singletons in one line of code (KAD 27)
date: 2019-09-11 10:54:25
cover: http://qiniu.zzzz1997.com/objects-kotlin/objects-kotlin.jpg
categories: 
- 技术
tags: kotlin
---

by Antonio Leiva
原文链接：[https://antonioleiva.com/objects-kotlin/](https://antonioleiva.com/objects-kotlin/)

<!--more-->

Kotlin objects are another element of the language that we Android developers are not familiarized with, because there is nothing like that in Java.

In fact, **an object is just a data type with a single implementation.** So if we want to find something similar in Java, that would be the Singleton pattern. We’ll compare them in the next section.

## Singleton vs object

Singleton in Java isn’t as easy to implement as it sounds. One might think it’s as simple as this:

```kotlin
public class Singleton {

    private Singleton() {
    }

    private static Singleton instance;

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }

        return instance;
    }
}
```

But this code is dangerous, especially if it’s used in different threads. If two threads access this singleton at a time, **two instances of this object could be generated.** A safer code would be:

```kotlin
public class Singleton {

    private static Singleton instance = null;

    private Singleton(){
    }

    private synchronized static void createInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
    }

    public static Singleton getInstance() {
        if (instance == null) createInstance();
        return instance;
    }
}
```

As you can see, you need a few lines to create a valid Singleton.

What would be the equivalent in **Kotlin**?

```kotlin
object Singleton
```

You don’t need more. You could use the *Show bytecode* tool you can find at *Tools -> Kotlin*, and then use the *Decompile* option. That way, you can see what’s the implementation that the Kotlin team decided to use for singletons.

I really recommend using that tool when you’re not sure what’s happening behind the scenes.

## Object declaration

Declaring an object is as simple as declaring a class.

Let’s declare for example an object that implements a database helper:

```kotlin
object MySQLOpenHandler : SQLiteOpenHelper(App.instance, "MyDB", null, 1) {

    override fun onCreate(db: SQLiteDatabase?) {
    }

    override fun onUpgrade(db: SQLiteDatabase?, oldVersion: Int, newVersion: Int) {
    }
}
```

As you can see, you just need to use the reserved word `object` instead of `class` and the rest is the same. Just take into account that **objects can’t have a constructor**, as we don’t call any constructors to access to them.

**The instance of the object will be created the first time we use it**. So there’s a lazy instantiation here: if an object is never used, the instance will never be created.

## Companion Object

Every class can implement a companion object, which is an object that is common to all instances of that class. It’d come to be similar to static fields in Java.

An implementation example:

```kotlin
class App : Application() {
    companion object {
        lateinit var instance: App
            private set
    }

    override fun onCreate() {
        super.onCreate()
        instance = this
    }
}
```

In this case I’m creating a class that extends `Application` and stores it in the `companion object` its unique instance.

The `lateinit` indicates that this property won’t have value from the beginning, but will be assigned before it’s used (otherwise it’ll throw an exception).

The `private set` is used so that a value can’t be assigned from an external class.

>Note: You may think that it’d make a lot of sense for App to be an object instead of a class. And so it is, but due to the way the Android framework instantiate the classes, if you try, you’ll see that the application throws an exception when it’s launched. You’ll need App to be a class, and you can create this little Singleton if you want to access to it.

## Object expressions

Objects can also be used to create anonymous class implementations.

An example:

```kotlin
recycler.adapter = object : RecyclerView.Adapter() {
    override fun onBindViewHolder(holder: RecyclerView.ViewHolder?, position: Int) {
    }

    override fun getItemCount(): Int {
    }

    override fun onCreateViewHolder(parent: ViewGroup?, viewType: Int): RecyclerView.ViewHolder {
    }

}
```

Every time you want to create an inline implementation of an interface, for instance, or extend another class, you’ll use the above notation.

But an object can also represent a class that doesn’t exist. You can create it on the fly:

```kotlin
val newObj = object {
    var x = "a"
    var y = "b"
}

Log.d(tag, "x:${newObj.x}, y:${newObj.y}")
```

## Conclusion

Objects are a new concept for those of us coming from Java 6, but there are many ideas that can be associated with existing ones, so you’ll get fast with them.
