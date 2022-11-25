# Понимание JVM
***
_Код для исследования_
***
```
public class JvmComprehension { 
    public static void main(String[] args) { 
        int i = 1;                      // 1 
        Object o = new Object();        // 2
        Integer ii = 2;                 // 3
        printAll(o, i, ii);             // 4
        System.out.println("finished"); // 7
    }
    private static void printAll(Object o, int i, Integer ii) {
        Integer uselessVar = 700;                   // 5
        System.out.println(o.toString() + i + ii);  // 6
    }
}
```
***
*_Компиляция кода в байт-код_*

В терминале команда javac JvmComprehension.java. Результат: файл JvmComprehension.class

*_Запуск программы_*

Улитита java стартует JVM, которая запустит класс filename.class
В терминале команда java JvmComprehension.

*_Загрузка классов (Classloader)_*
  
    Иерархия загрузчиков классов
![ClassLoader Hierarchy](http://www.thistechnologylife.com/wp-content/uploads/2018/07/Class-Loader-600x294.png)
- Bootstrap: загружает стандартные классы JDK из .jar
- Platform ClassLoader: загружает классы расширений (java.util)
- Application ClassLoader: загружает JvmComprehension.class
- User's Classloader: в примере не предусмотрен.

Если класс не найден, выбрасывается исключение ClassNotFoundException.

*_Связывание (Linking)_*

- Verify: проверка корректности байт-кода;
- Prepare: выделение оперативной памяти под статические поля и инициализация их значениями по умолчанию (без явной инициализации!!!);
- Resolve: разрешение символьных ссылок типов, полей, методов.

*_Инициализация объекта (Initialization)_*

#### *Построчный анализ кода*

```
public class JvmComprehension {  // класс попадает в область памяти Metaspace    

    public static void main(String[] args) { // в стеке создается фрейм main()
       
        int i = 1;                      // 1. в фрейме main() создается примитивная переменная i
       
        Object o = new Object();        // 2. в куче (heap) создается новый (new) объект (Object()), 
        в стеке создается переменная о, содержащая ссылку на данный объект.
        
        Integer ii = 2;                 // 3. в куче создается объект класса Integer со значением 2, 
        в стеке переменная ii, содержащая ссылку на объект
        
        printAll(o, i, ii);             // 4. в стеке создается фрейм printAll(), в котором создаются переменная, 
        хранящая ссылку на объект Object() (стр.2), переменная i, 
        в которую передано значение переменной i (стр.1) и 
        переменная ii, которая приняла ссылку на объект Integer (стр.3)
        
        System.out.println("finished"); // 7. в стеке создается фрейм println(), 
        ссылочная переменная типа String создается в фрейме println(), эта переменная содержит ссылку 
        на место в куче, хранящее значение "finished", 
    }
    
    private static void printAll(Object o, int i, Integer ii) {
        
        Integer uselessVar = 700;                   // 5. в куче создается объект Integer c значанием 700, переменная uselessVar, 
        хранящая ссылку на этот объект, создается в стеке printAll()
        
        System.out.println(o.toString() + i + ii);  // 6. в стеке создается новый фрейм println(). 
        При вызове метода toString() у объекта о создается новый фрейм toString(), 
        куда передается ссылка на объект из кучи, созданный в стр.2., переменные i и ii с соответствующими значениям. 
        После выполнения метода фрейм toSting() уничтожается, а полученный результат записывается в кучу 
        во вновь созданный объект класса String. Ссылка на этот объект записывается в статическое поле out фрейма println().
    }
}
```
*_Сборщик мусора_*

Запускается с полной приостановкой выполнения программы и определяет неиспользуемые объекты путем обхода графа достижимых объектов.
