# item16. public 클래스에서는 public 필드가 아닌 접근자 메소드를 사용하라.

```java
class Point{
	public double x;
	public double y;
}
```

* **API의 필드를 수정하지 않고는 내부 표현을 바꿀 수 없다.**
  * getter/setter 메서드가 존재한다면 얼마든지 표현 변경이 가능하다.
* **불변식을 보장할 수 없다.**
  * 클라이언트가 직접 데이터를 변경할 수 있다.
* **외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없다.**
  * 1차원적인 접근만 가능하고, 추가 로직을 삽입할 수 없다.

철저한 객체 지향 프로그래머는 필드를 모두 private으로 바꾸고 public getter를 추가한다.

```java
class Point {
        private double x;
        private double y;

        public Point(double x, double y) {
            this.x = x;
            this.y = y;
        }

        public double getX() {
            return x;
        }

        public void setX(double x) {
            this.x = x;
        }

        public double getY() {
            return y;
        }

        public void setY(double y) {
            this.y = y;
        }
    }
```

public 클래스가 필드를 공개하면 이를 사용하는 클라이언트가 생겨날 것이므로 내부 표현 방식을 마음대로 바꿀 수 없게 된다.

—> 아 이게 사용하는 입장에서 얘기하는 게 아니라, 클래스를 제공한 입장에서의 얘기구나.

package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없다. (접근자 방법보다 훨씬 깔끔하기도 함)

* **package-private 클래스에서 public 데이터 필드 노출**: 클라이언트 코드가 내부 표현에 묶이기는 하나, 클라이언트도 어차피 이 클래스를 포함하는 패키지 안에서만 동작하는 코드일 뿐이다. 따라서 패키지 바깥 코드는 전혀 손대지 않고도 데이터 표현 방식을 바꿀 수 있다.
* **private 중첩 클래스에서 public 데이터 필드 노출**: 이 경우는 수정 범위가 더 좁아져서 이 클래스를 포함하는 외부 클래스까지로 제한된다.

자바 플랫폼 라이브러리에도 public 클래스의 필드를 직접 노출하지 말라는 규칙을 어기는 사례가 종종 있다. 대표적인 예로 `java.awt` 패키지의 `Point`와 `Dimension`클래스이다.

* item67에서 설명하듯, 내부를 노출한 Dimension 클래스의 심각한 성능 문제는 오늘까지도 해결되지 못했다.
*   참고 (Point , Dimension 클래스)

    ```java

    package java.awt;

    import java.awt.geom.Point2D;
    import java.beans.Transient;

    public class Point extends Point2D implements java.io.Serializable {
        /**
         * The X coordinate of this <code>Point</code>.
         * If no X coordinate is set it will default to 0.
         *
         * @serial
         * @see #getLocation()
         * @see #move(int, int)
         * @since 1.0
         */
        public int x;

        /**
         * The Y coordinate of this <code>Point</code>.
         * If no Y coordinate is set it will default to 0.
         *
         * @serial
         * @see #getLocation()
         * @see #move(int, int)
         * @since 1.0
         */
        public int y;

        /*
         * JDK 1.1 serialVersionUID
         */
        private static final long serialVersionUID = -5276940640259749850L;

        /**
         * Constructs and initializes a point at the origin
         * (0,&nbsp;0) of the coordinate space.
         * @since       1.1
         */
        public Point() {
            this(0, 0);
        }

        /**
         * Constructs and initializes a point with the same location as
         * the specified <code>Point</code> object.
         * @param       p a point
         * @since       1.1
         */
        public Point(Point p) {
            this(p.x, p.y);
        }

        /**
         * Constructs and initializes a point at the specified
         * {@code (x,y)} location in the coordinate space.
         * @param x the X coordinate of the newly constructed <code>Point</code>
         * @param y the Y coordinate of the newly constructed <code>Point</code>
         * @since 1.0
         */
        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }

        /**
         * {@inheritDoc}
         * @since 1.2
         */
        public double getX() {
            return x;
        }

        /**
         * {@inheritDoc}
         * @since 1.2
         */
        public double getY() {
            return y;
        }

        /**
         * Returns the location of this point.
         * This method is included for completeness, to parallel the
         * <code>getLocation</code> method of <code>Component</code>.
         * @return      a copy of this point, at the same location
         * @see         java.awt.Component#getLocation
         * @see         java.awt.Point#setLocation(java.awt.Point)
         * @see         java.awt.Point#setLocation(int, int)
         * @since       1.1
         */
        @Transient
        public Point getLocation() {
            return new Point(x, y);
        }

        /**
         * Sets the location of the point to the specified location.
         * This method is included for completeness, to parallel the
         * <code>setLocation</code> method of <code>Component</code>.
         * @param       p  a point, the new location for this point
         * @see         java.awt.Component#setLocation(java.awt.Point)
         * @see         java.awt.Point#getLocation
         * @since       1.1
         */
        public void setLocation(Point p) {
            setLocation(p.x, p.y);
        }

        /**
         * Changes the point to have the specified location.
         * <p>
         * This method is included for completeness, to parallel the
         * <code>setLocation</code> method of <code>Component</code>.
         * Its behavior is identical with <code>move(int,&nbsp;int)</code>.
         * @param       x the X coordinate of the new location
         * @param       y the Y coordinate of the new location
         * @see         java.awt.Component#setLocation(int, int)
         * @see         java.awt.Point#getLocation
         * @see         java.awt.Point#move(int, int)
         * @since       1.1
         */
        public void setLocation(int x, int y) {
            move(x, y);
        }

        /**
         * Sets the location of this point to the specified double coordinates.
         * The double values will be rounded to integer values.
         * Any number smaller than <code>Integer.MIN_VALUE</code>
         * will be reset to <code>MIN_VALUE</code>, and any number
         * larger than <code>Integer.MAX_VALUE</code> will be
         * reset to <code>MAX_VALUE</code>.
         *
         * @param x the X coordinate of the new location
         * @param y the Y coordinate of the new location
         * @see #getLocation
         */
        public void setLocation(double x, double y) {
            this.x = (int) Math.floor(x+0.5);
            this.y = (int) Math.floor(y+0.5);
        }

        /**
         * Moves this point to the specified location in the
         * {@code (x,y)} coordinate plane. This method
         * is identical with <code>setLocation(int,&nbsp;int)</code>.
         * @param       x the X coordinate of the new location
         * @param       y the Y coordinate of the new location
         * @see         java.awt.Component#setLocation(int, int)
         */
        public void move(int x, int y) {
            this.x = x;
            this.y = y;
        }

        /**
         * Translates this point, at location {@code (x,y)},
         * by {@code dx} along the {@code x} axis and {@code dy}
         * along the {@code y} axis so that it now represents the point
         * {@code (x+dx,y+dy)}.
         *
         * @param       dx   the distance to move this point
         *                            along the X axis
         * @param       dy    the distance to move this point
         *                            along the Y axis
         */
        public void translate(int dx, int dy) {
            this.x += dx;
            this.y += dy;
        }

        /**
         * Determines whether or not two points are equal. Two instances of
         * <code>Point2D</code> are equal if the values of their
         * <code>x</code> and <code>y</code> member fields, representing
         * their position in the coordinate space, are the same.
         * @param obj an object to be compared with this <code>Point2D</code>
         * @return <code>true</code> if the object to be compared is
         *         an instance of <code>Point2D</code> and has
         *         the same values; <code>false</code> otherwise.
         */
        public boolean equals(Object obj) {
            if (obj instanceof Point) {
                Point pt = (Point)obj;
                return (x == pt.x) && (y == pt.y);
            }
            return super.equals(obj);
        }

        /**
         * Returns a string representation of this point and its location
         * in the {@code (x,y)} coordinate space. This method is
         * intended to be used only for debugging purposes, and the content
         * and format of the returned string may vary between implementations.
         * The returned string may be empty but may not be <code>null</code>.
         *
         * @return  a string representation of this point
         */
        public String toString() {
            return getClass().getName() + "[x=" + x + ",y=" + y + "]";
        }
    }
    ```

    ```java
    package java.awt;

    import java.awt.geom.Dimension2D;
    import java.beans.Transient;

    public class Dimension extends Dimension2D implements java.io.Serializable {

        /**
         * The width dimension; negative values can be used.
         *
         * @serial
         * @see #getSize
         * @see #setSize
         * @since 1.0
         */
        public int width;

        /**
         * The height dimension; negative values can be used.
         *
         * @serial
         * @see #getSize
         * @see #setSize
         * @since 1.0
         */
        public int height;

        /*
         * JDK 1.1 serialVersionUID
         */
         private static final long serialVersionUID = 4723952579491349524L;

        /**
         * Initialize JNI field and method IDs
         */
        private static native void initIDs();

        static {
            /* ensure that the necessary native libraries are loaded */
            Toolkit.loadLibraries();
            if (!GraphicsEnvironment.isHeadless()) {
                initIDs();
            }
        }

        /**
         * Creates an instance of <code>Dimension</code> with a width
         * of zero and a height of zero.
         */
        public Dimension() {
            this(0, 0);
        }

        /**
         * Creates an instance of <code>Dimension</code> whose width
         * and height are the same as for the specified dimension.
         *
         * @param    d   the specified dimension for the
         *               <code>width</code> and
         *               <code>height</code> values
         */
        public Dimension(Dimension d) {
            this(d.width, d.height);
        }

        /**
         * Constructs a <code>Dimension</code> and initializes
         * it to the specified width and specified height.
         *
         * @param width the specified width
         * @param height the specified height
         */
        public Dimension(int width, int height) {
            this.width = width;
            this.height = height;
        }

        /**
         * {@inheritDoc}
         * @since 1.2
         */
        public double getWidth() {
            return width;
        }

        /**
         * {@inheritDoc}
         * @since 1.2
         */
        public double getHeight() {
            return height;
        }

        /**
         * Sets the size of this <code>Dimension</code> object to
         * the specified width and height in double precision.
         * Note that if <code>width</code> or <code>height</code>
         * are larger than <code>Integer.MAX_VALUE</code>, they will
         * be reset to <code>Integer.MAX_VALUE</code>.
         *
         * @param width  the new width for the <code>Dimension</code> object
         * @param height the new height for the <code>Dimension</code> object
         * @since 1.2
         */
        public void setSize(double width, double height) {
            this.width = (int) Math.ceil(width);
            this.height = (int) Math.ceil(height);
        }

        /**
         * Gets the size of this <code>Dimension</code> object.
         * This method is included for completeness, to parallel the
         * <code>getSize</code> method defined by <code>Component</code>.
         *
         * @return   the size of this dimension, a new instance of
         *           <code>Dimension</code> with the same width and height
         * @see      java.awt.Dimension#setSize
         * @see      java.awt.Component#getSize
         * @since    1.1
         */
        @Transient
        public Dimension getSize() {
            return new Dimension(width, height);
        }

        /**
         * Sets the size of this <code>Dimension</code> object to the specified size.
         * This method is included for completeness, to parallel the
         * <code>setSize</code> method defined by <code>Component</code>.
         * @param    d  the new size for this <code>Dimension</code> object
         * @see      java.awt.Dimension#getSize
         * @see      java.awt.Component#setSize
         * @since    1.1
         */
        public void setSize(Dimension d) {
            setSize(d.width, d.height);
        }

        /**
         * Sets the size of this <code>Dimension</code> object
         * to the specified width and height.
         * This method is included for completeness, to parallel the
         * <code>setSize</code> method defined by <code>Component</code>.
         *
         * @param    width   the new width for this <code>Dimension</code> object
         * @param    height  the new height for this <code>Dimension</code> object
         * @see      java.awt.Dimension#getSize
         * @see      java.awt.Component#setSize
         * @since    1.1
         */
        public void setSize(int width, int height) {
            this.width = width;
            this.height = height;
        }

        /**
         * Checks whether two dimension objects have equal values.
         */
        public boolean equals(Object obj) {
            if (obj instanceof Dimension) {
                Dimension d = (Dimension)obj;
                return (width == d.width) && (height == d.height);
            }
            return false;
        }

        /**
         * Returns the hash code for this <code>Dimension</code>.
         *
         * @return    a hash code for this <code>Dimension</code>
         */
        public int hashCode() {
            int sum = width + height;
            return sum * (sum + 1)/2 + width;
        }

        /**
         * Returns a string representation of the values of this
         * <code>Dimension</code> object's <code>height</code> and
         * <code>width</code> fields. This method is intended to be used only
         * for debugging purposes, and the content and format of the returned
         * string may vary between implementations. The returned string may be
         * empty but may not be <code>null</code>.
         *
         * @return  a string representation of this <code>Dimension</code>
         *          object
         */
        public String toString() {
            return getClass().getName() + "[width=" + width + ",height=" + height + "]";
        }
    }
    ```

public 클래스의 필드가 불변이라면 직접 노출할 때의 단점이 조금은 줄어들지만, 여전히 결코 좋은 생각이 아니다. API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점은 여전하다. 단, 불변식은 보장할 수 있게 된다.

```java
public class Time {
  private static final int HOURS_PER_DAY = 24;
  private static final int MINUTES_PER_HOUR = 60;
  
  public final int hour;
  public final int minute;
  
  public Time (int hour, int minute) { // 각 인스턴스가 유효한 시간을 표현함을 보장.
      if (hour < 0 || hour >= HOURS_PER_DAY) 
          throw new IllegalArgumentException("시간: " + hour);
      if (minute < 0 || minute >= MINUTES_PER_HOUR)
          throw new IllegalArgumentException("분: " + minute);
      this.hour = hour;
      this.minute = minute;
  }
	// 나머지 코드 생략
}
```

{% hint style="info" %}
<mark style="background-color:yellow;">**💡 public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다. 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다. 하지만 package-private 클래스나 private 중첩 클래스에서은 종종 (불변이든 가변이든) 필드를 노출하는 편이 나을 때도 있다.**</mark>
{% endhint %}
