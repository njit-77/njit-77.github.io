---
layout:     post
title:      c#Equal函数and运算符'=='
category: 	blog
---

1、==、!=、<、>、<= 和>= 运算符为比较运算符(comparison operator)。C#语言规范5.0中文版中比较运算符的描述如下：
![1](/images/c%23Equal函数and运算符'%3D%3D'/1.PNG)

2、通用类型系统
![2](/images/c%23Equal函数and运算符'%3D%3D'/2.PNG)

3、值类型Equal函数 and 运算符'=='
3.1、常见类型 int、float、double、decimal等虽然继承自ValueType，但其结构体内部重写了Equal。
3.1.1、 int，float，double，decimal内部的Equal函数和 '=='重载符函数。

```c#
Int32
{
    public override bool Equals(Object obj)
    {
        if (!(obj is Int32)) 
        {
            return false;
        }
        return m_value == ((Int32)obj).m_value;
    }
    
    [System.Runtime.Versioning.NonVersionable]
    public bool Equals(Int32 obj)
    {
        return m_value == obj;
    }
}

Double
{
    // True if obj is another Double with the same value as the current instance.  This is a method of object equality, that only returns true if obj is also a double.
    public override bool Equals(Object obj) 
    {
        if (!(obj is Double)) 
        {
            return false;
        }
        double temp = ((Double)obj).m_value;
        
        // This code below is written this way for performance reasons i.e the != and == check is intentional.
        if (temp == m_value) 
        {
            return true;
        }
        return IsNaN(temp) && IsNaN(m_value);
    }
    
    public bool Equals(Double obj)
    {
        if (obj == m_value) 
        {
            return true;
        }
        return IsNaN(obj) && IsNaN(m_value);
    }
    
    [System.Runtime.Versioning.NonVersionable]
    public static bool operator ==(Double left, Double right) 
    {
        return left == right;
    }
}

Single
{
    public override bool Equals(Object obj) 
    {
        if (!(obj is Single)) 
        {
            return false;
        }
        float temp = ((Single)obj).m_value;
        if (temp == m_value) 
        {
            return true;
        }
        return IsNaN(temp) && IsNaN(m_value);
    }
    
    public bool Equals(Single obj)
    {
        if (obj == m_value) 
        {
            return true;
        }
        return IsNaN(obj) && IsNaN(m_value);
    }
    
    [System.Runtime.Versioning.NonVersionable]
    public static bool operator ==(Single left, Single right) 
    {
        return left == right;
    }		
}

Decimal
{
    // Checks if this Decimal is equal to a given object. Returns true if the given object is a boxed Decimal and its value is equal to the value of this Decimal. Returns false otherwise.
    [System.Security.SecuritySafeCritical]  // auto-generated
    public override bool Equals(Object value) 
    {
        if (value is Decimal) 
        {
            Decimal other = (Decimal)value;
            return FCallCompare(ref this, ref other) == 0;
        }
        return false;
    }
    
    [System.Security.SecuritySafeCritical]  // auto-generated
    public bool Equals(Decimal value)
    {
        return FCallCompare(ref this, ref value) == 0;
    }
    
    [System.Security.SecuritySafeCritical]  // auto-generated
    public static bool operator ==(Decimal d1, Decimal d2) 
    {
        return FCallCompare(ref d1, ref d2) == 0;
    }
    
    //暂时不知道此函数内部代码，如有知道还望告知。根据测试结果，推测如果两个decimal数相等，返回0
    [System.Security.SecurityCritical]  // auto-generated
    [ResourceExposure(ResourceScope.None)]
    [MethodImplAttribute(MethodImplOptions.InternalCall)]
    [ReliabilityContract(Consistency.WillNotCorruptState, Cer.Success)]
    private static extern int FCallCompare(ref Decimal d1, ref Decimal d2);	
}
```

3.1.2、感兴趣的可去[Reference Source ](http://referencesource.microsoft.com/#mscorlib)查看全部代码。
3.1.3、测试代码：

```c#
//T is int 、float、double、decimal、byte、char
T a = 1234567890;//0.1234567890f、0.123456789、1234567890M、(byte)11、'a'
T b = 1234567890;//0.1234567890f、0.123456789、1234567890M、(byte)11、'a'
    
Console.WriteLine(a == b);//返回true
Console.WriteLine(a.Equals(b));//返回true
Console.WriteLine(a.Equals((object)b));//返回true
			
/*
    Console.WriteLine((object)a == b);//编译错误：运算符‘==’无法应用与‘object’和‘T’类型操作数
    Console.WriteLine(a == (object)b);//编译错误：运算符‘==’无法应用与‘object’和‘T’类型操作数
    //Console.WriteLine((object)a == (object)b);//返回false，下面解释为什么是false。这个是引用类型'=='，放到下文介绍
*/
```

3.1.4、结论：对于简单常见值类型 int、float、double、decimal等，Equal函数 and 运算符'=='，如果其值相等，返回true；否则，返回false。

3.2、 结构体struct
3.2.1、 ValueType内部的Equals函数

```c#
		ValueType
		{
	        [System.Security.SecuritySafeCritical]
	        public override bool Equals (Object obj) {
	            BCLDebug.Perf(false, "ValueType::Equals is not fast.  "+this.GetType().FullName+" should override Equals(Object)");
	            if (null==obj) {
	                return false;
	            }
	            RuntimeType thisType = (RuntimeType)this.GetType();
	            RuntimeType thatType = (RuntimeType)obj.GetType();
	 
	            if (thatType!=thisType) {
	                return false;
	            }
	 
	            Object thisObj = (Object)this;
	            Object thisResult, thatResult;
	 
	            // if there are no GC references in this object we can avoid reflection 
	            // and do a fast memcmp
	            if (CanCompareBits(this))
	                return FastEqualsCheck(thisObj, obj);
	 
	            FieldInfo[] thisFields = thisType.GetFields(BindingFlags.Instance | BindingFlags.Public | BindingFlags.NonPublic);
	 
	            for (int i=0; i<thisFields.Length; i++) {
	                thisResult = ((RtFieldInfo)thisFields[i]).UnsafeGetValue(thisObj);
	                thatResult = ((RtFieldInfo)thisFields[i]).UnsafeGetValue(obj);
	                
	                if (thisResult == null) {
	                    if (thatResult != null)
	                        return false;
	                }
	                else
	                if (!thisResult.Equals(thatResult)) {
	                    return false;
	                }
	            }
	 
	            return true;
	        }
	 
	        [System.Security.SecuritySafeCritical]  // auto-generated
	        [ResourceExposure(ResourceScope.None)]
	        [MethodImplAttribute(MethodImplOptions.InternalCall)]
	        private static extern bool CanCompareBits(Object obj);
	 
	        [System.Security.SecuritySafeCritical]  // auto-generated
	        [ResourceExposure(ResourceScope.None)]
	        [MethodImplAttribute(MethodImplOptions.InternalCall)]
	        private static extern bool FastEqualsCheck(Object a, Object b);			
		}
```

3.2.2、结构体(只有值类型，重写Equal函数 and 运算符'==')
3.2.2.1、测试代码：

```c#
    struct Point
    {
        public double x;
        public double y;
        public double z;

        public Point(double X, double Y, double Z)
        {
            this.x = X;
            this.y = Y;
            this.z = Z;
        }

        public override bool Equals(Object obj)
        {
            if (!(obj is Point))
            {
                return false;
            }

            if (((Point)obj).x == this.x)
            {
                return true;
            }

            return false;
        }
        public bool Equals(Point obj)
        {
            if (obj.x == this.x)
            {
                return true;
            }

            return false;
        }
        
		//运算符“Point.operator ==(Point, Point)”要求也要定义匹配的运算符“!=”
        public static bool operator ==(Point left, Point right)
        {
            return left.x == right.x;
        }

        public static bool operator !=(Point left, Point right)
        {
            return left.x != right.x;
        }
    }

    Point p1 = new Point(1, 2, 3);
    Point p2 = p1;
    
    p1.y = 100;
    Console.WriteLine(p1 == p2);//返回true
    Console.WriteLine(p1.Equals(p2)); // 返回true
    Console.WriteLine(p1.Equals((object)p2)); // 返回true
```

3.2.2.2、结论：此时程序执行我们重写的Equal函数 and 运算符'=='。

3.2.3、结构体(只有值类型，不重写Equal函数 and 运算符'==')
3.2.3.1、测试代码：

```c#
    struct Point
    {
        public double x;
        public double y;
        public double z;

        public Point(double X, double Y, double Z)
        {
            this.x = X;
            this.y = Y;
            this.z = Z;
        }
    }

    Point p1 = new Point(1, 2, 3);
    Point p2 = p1;

    Console.WriteLine(p1 == p2);//编译错误：运算符"=="无法应用于"Point"和"Point"类型的操作数
    Console.WriteLine(p1.Equals(p2)); // 返回true
    Console.WriteLine(p1.Equals((object)p2)); // 返回true
    p1.y = 100;
    Console.WriteLine(p1.Equals(p2)); // 返回false
    Console.WriteLine(p1.Equals((object)p2)); // 返回false
```

3.2.3.2、程序执行时，CanCompareBits(this)返回true，代码执行return FastEqualsCheck(thisObj, obj);
3.2.3.3、结论：程序判断struct里面所有字段的值，如果全部相等，返回true；否则，返回false。

3.2.4、复杂结构体(有值类型、引用类型，重写Equal函数 and 运算符'==')
3.2.4.1、测试代码：

```c#
    public struct ValPoint
    {
        public double x;
        public double y;
        public double z;

        public ValPoint(double X, double Y, double Z)
        {
            this.x = X;
            this.y = Y;
            this.z = Z;
        }
        
        public static bool operator ==(ValPoint left, ValPoint right)
        {
            return left.x == right.x;
        }

        public static bool operator !=(ValPoint left, ValPoint right)
        {
            return left.x != right.x;
        }
    }

    public class RefPoint
    {
        public double x;
        public double y;
        public double z;

        public RefPoint(double X, double Y, double Z)
        {
            this.x = X;
            this.y = Y;
            this.z = Z;
        }
    }

    public struct ValLine
    {
        public ValPoint vPoint;       // 值类型成员

        public RefPoint rPoint;       // 引用类型成员

        public ValLine(ValPoint vPoint, RefPoint rPoint)
        {
            this.vPoint = vPoint;
            this.rPoint = rPoint;
        }

        public override bool Equals(Object obj)
        {
            if (!(obj is ValLine))
            {
                return false;
            }

            if (((ValLine)obj).vPoint == this.vPoint)
            {
                return true;
            }

            return false;
        }

        public bool Equals(ValLine obj)
        {
            if (obj.vPoint == this.vPoint)
            {
                return true;
            }

            return false;
        }

        public static bool operator ==(ValLine left, ValLine right)
        {
            return left.vPoint == right.vPoint;
        }

        public static bool operator !=(ValLine left, ValLine right)
        {
            return left.vPoint != right.vPoint;
        }
    }


    ValPoint vPoint = new ValPoint(1, 2, 3);
    ValPoint vPoint2 = new ValPoint(1, 2, 3);
    ValPoint vPoint3 = new ValPoint(10, 20, 30);
    RefPoint rPoint = new RefPoint(4, 5, 6);
    RefPoint rPoint2 = new RefPoint(7, 8, 9);

    ValLine p1 = new ValLine(vPoint, rPoint);
    ValLine p2 = p1;

    p2.vPoint = vPoint2;
    Console.WriteLine(p1 == p2); //返回true
    Console.WriteLine(p1.Equals(p2)); //返回true
    Console.WriteLine(p1.Equals((object)p2)); //返回true

    p2 = p1;
    p2.vPoint = vPoint3;
    Console.WriteLine(p1 == p2); //返回true
    Console.WriteLine(p1.Equals(p2)); //返回false
    Console.WriteLine(p1.Equals((object)p2)); //返回false

    p2 = p1;
    p2.rPoint = rPoint2;
    Console.WriteLine(p1 == p2); //返回true
    Console.WriteLine(p1.Equals(p2)); //返回true
    Console.WriteLine(p1.Equals((object)p2)); //返回true
```

3.2.4.2、结论：此时程序执行我们重写的Equal函数 and 运算符'=='。

3.2.5、复杂结构体(内部值类型、引用类型，不重写Equal函数 and 运算符'==')
3.2.5.1、测试代码：

```c#
    public struct ValPoint
    {
        public double x;
        public double y;
        public double z;

        public ValPoint(double X, double Y, double Z)
        {
            this.x = X;
            this.y = Y;
            this.z = Z;
        }
    }

    public class RefPoint
    {
        public double x;
        public double y;
        public double z;

        public RefPoint(double X, double Y, double Z)
        {
            this.x = X;
            this.y = Y;
            this.z = Z;
        }
    }

    public struct ValLine
    {
        public ValPoint vPoint;       // 值类型成员

        public RefPoint rPoint;       // 引用类型成员

        public ValLine(ValPoint vPoint, RefPoint rPoint)
        {
            this.vPoint = vPoint;
            this.rPoint = rPoint;
        }
    }


    ValPoint vPoint = new ValPoint(1, 2, 3);
    ValPoint vPoint2 = new ValPoint(1, 2, 3);
    ValPoint vPoint3 = new ValPoint(10, 20, 30);
    RefPoint rPoint = new RefPoint(4, 5, 6);
    RefPoint rPoint2 = new RefPoint(7, 8, 9);

    ValLine p1 = new ValLine(vPoint, rPoint);
    ValLine p2 = p1;

	Console.WriteLine(p1 == p2);//编译错误：运算符"=="无法应用于"Point"和"Point"类型的操作数

    p2.vPoint = vPoint2;
    Console.WriteLine(p1.Equals(p2)); //返回true
    Console.WriteLine(p1.Equals((object)p2)); //返回true

	p2 = p1;
	p2.vPoint = vPoint3;
    Console.WriteLine(p1.Equals(p2)); //返回false
    Console.WriteLine(p1.Equals((object)p2)); //返回false
	
	p2 = p1;
    p2.rPoint = rPoint2;
    Console.WriteLine(p1.Equals(p2)); //返回false
    Console.WriteLine(p1.Equals((object)p2)); //返回false
```

3.2.5.2、程序执行时，CanCompareBits(this)返回false，代码执行ValueType类Equal函数的下面语句

```c#
	            FieldInfo[] thisFields = thisType.GetFields(BindingFlags.Instance | BindingFlags.Public | BindingFlags.NonPublic);
	 
	            for (int i=0; i<thisFields.Length; i++) {
	                thisResult = ((RtFieldInfo)thisFields[i]).UnsafeGetValue(thisObj);
	                thatResult = ((RtFieldInfo)thisFields[i]).UnsafeGetValue(obj);
	                
	                if (thisResult == null) {
	                    if (thatResult != null)
	                        return false;
	                }
	                else
	                if (!thisResult.Equals(thatResult)) {
	                    return false;
	                }
	            }
	 
	            return true;
```

3.2.5.3、结论：程序判断struct里面所有字段,值类型就判断值是否相等；引用类型就判断是否引用相等。

4、引用类型Equal函数 and 运算符'=='
4.1、字符串string
4.1.1、C#语言规范5.0中文版中的字符串相等运算符介绍
![3](/images/c%23Equal函数and运算符'%3D%3D'/3.PNG)
4.1.2、string的Equal函数和'=='重载运算符函数代码

```c#
		String
		{
	        // Determines whether two strings match.
	        [ReliabilityContract(Consistency.WillNotCorruptState, Cer.MayFail)]
	        public override bool Equals(Object obj) {
	            if (this == null)                        //this is necessary to guard against reverse-pinvokes and
	                throw new NullReferenceException();  //other callers who do not use the callvirt instruction
	 
	            String str = obj as String;
	            if (str == null)
	                return false;
	 
	            if (Object.ReferenceEquals(this, obj))
	                return true;
	 
	            if (this.Length != str.Length)
	                return false;
	 
	            return EqualsHelper(this, str);
	        }
	 
	        // Determines whether two strings match.
	        [Pure]
	        [ReliabilityContract(Consistency.WillNotCorruptState, Cer.MayFail)]
	        public bool Equals(String value) {
	            if (this == null)                        //this is necessary to guard against reverse-pinvokes and
	                throw new NullReferenceException();  //other callers who do not use the callvirt instruction
	 
	            if (value == null)
	                return false;
	 
	            if (Object.ReferenceEquals(this, value))
	                return true;
	            
	            if (this.Length != value.Length)
	                return false;
	 
	            return EqualsHelper(this, value);
	        }

	        public static bool operator == (String a, String b) {
	           return String.Equals(a, b);
	        }

	        // Determines whether two Strings match.
	        [Pure]
	        public static bool Equals(String a, String b) {
	            if ((Object)a==(Object)b) {
	                return true;
	            }
	 
	            if ((Object)a==null || (Object)b==null) {
	                return false;
	            }
	 
	            if (a.Length != b.Length)
	                return false;
	 
	            return EqualsHelper(a, b);
	        }

	        [System.Security.SecuritySafeCritical]  // auto-generated
	        [ReliabilityContract(Consistency.WillNotCorruptState, Cer.MayFail)]
	        private unsafe static bool EqualsHelper(String strA, String strB)
	        {
	            Contract.Requires(strA != null);
	            Contract.Requires(strB != null);
	            Contract.Requires(strA.Length == strB.Length);
	 
	            int length = strA.Length;
	 
	            fixed (char* ap = &strA.m_firstChar) fixed (char* bp = &strB.m_firstChar)
	            {
	                char* a = ap;
	                char* b = bp;
	 
	                // unroll the loop
	#if AMD64
	                // for AMD64 bit platform we unroll by 12 and
	                // check 3 qword at a time. This is less code
	                // than the 32 bit case and is shorter
	                // pathlength
	 
	                while (length >= 12)
	                {
	                    if (*(long*)a     != *(long*)b) return false;
	                    if (*(long*)(a+4) != *(long*)(b+4)) return false;
	                    if (*(long*)(a+8) != *(long*)(b+8)) return false;
	                    a += 12; b += 12; length -= 12;
	                }
	#else
	                while (length >= 10)
	                {
	                    if (*(int*)a != *(int*)b) return false;
	                    if (*(int*)(a+2) != *(int*)(b+2)) return false;
	                    if (*(int*)(a+4) != *(int*)(b+4)) return false;
	                    if (*(int*)(a+6) != *(int*)(b+6)) return false;
	                    if (*(int*)(a+8) != *(int*)(b+8)) return false;
	                    a += 10; b += 10; length -= 10;
	                }
	#endif
	 
	                // This depends on the fact that the String objects are
	                // always zero terminated and that the terminating zero is not included
	                // in the length. For odd string sizes, the last compare will include
	                // the zero terminator.
	                while (length > 0) 
	                {
	                    if (*(int*)a != *(int*)b) break;
	                    a += 2; b += 2; length -= 2;
	                }
	 
	                return (length <= 0);
	            }
	        }	        

		}

```

4.1.3、Object.ReferenceEquals(this, value)如果this、value是同一个引用，返回true；否则，返回false。

```c#
            {
                string a = "a1!";
                string b = "a1!";
                Console.WriteLine(Object.ReferenceEquals(a, b));//返回true,可以判断编译器将a与b所指向的"a1!"优化成一个地方。
            }

            {
                string a = "Test";
                string b = string.Copy(a);
                Console.WriteLine(Object.ReferenceEquals(a, b));//返回false
            }

            {
                string a = "Test";
                string b = (string)a.Clone();
                Console.WriteLine(Object.ReferenceEquals(a, b));//返回true
            }

            {
                char[] ch = new char[] { 'a', 'A', '@' };
                string a = "aA@";
                string b = new string(ch);
                Console.WriteLine(Object.ReferenceEquals(a, b));//返回false
            }

```

4.1.4、学习EqualsHelper(String strA, String strB)函数之前，我们先看一段代码

```c#
            unsafe
            {
                char[] firstCharA = "abc".ToCharArray();
                int length = firstCharA.Length;
                fixed (char* ap = firstCharA)
                {
                    for (int i = 0; i < length; i++)
                    {
                        Console.WriteLine(*(char*)(ap + i));
                    }
                }
            }

            unsafe
            {
                int[] firstCharA = new int[] { 1, 20, 300 };
                int length = firstCharA.Length;
                fixed (int* ap = firstCharA)
                {
                    for (int i = 0; i < length; i++)
                    {
                        Console.WriteLine(*(int*)(ap + i));
                    }
                }
            }            

```

![4](/images/c%23Equal函数and运算符'%3D%3D'/4.PNG)

4.1.5、修改后EqualsHelper(String strA, String strB)函数

```c#
        private unsafe static bool EqualsHelper(String strA, String strB)
        {
            Contract.Requires(strA != null);
            Contract.Requires(strB != null);
            Contract.Requires(strA.Length == strB.Length);

            int length = strA.Length;

            char[] firstCharA = strA.ToCharArray();
            char[] firstCharB = strB.ToCharArray();

            fixed (char* ap = &firstCharA[0]) fixed (char* bp = &firstCharB[0])//因无法使用m_firstChar，此处是我自行修改。ps：个人认为m_firstChar是指字符串的第一字符，但是无法证明。
            //fixed (char* ap = &strA.m_firstChar) fixed (char* bp = &strB.m_firstChar)
            {
                char* a = ap;
                char* b = bp;
                
                while (length >= 10)
                {
                    if (*(int*)a != *(int*)b) return false;
                    if (*(int*)(a + 2) != *(int*)(b + 2)) return false;
                    if (*(int*)(a + 4) != *(int*)(b + 4)) return false;
                    if (*(int*)(a + 6) != *(int*)(b + 6)) return false;
                    if (*(int*)(a + 8) != *(int*)(b + 8)) return false;
                    a += 10; b += 10; length -= 10;
                }

                // This depends on the fact that the String objects are
                // always zero terminated and that the terminating zero is not included
                // in the length. For odd string sizes, the last compare will include
                // the zero terminator.
                while (length > 0)
                {
                    if (*(int*)a != *(int*)b) break;
                    a += 2; b += 2; length -= 2;
                }

                return (length <= 0);
            }
        }

```

4.1.6、修改说明

```c#
1、fixed (char* ap = &strA.m_firstChar) fixed (char* bp = &strB.m_firstChar)-> fixed (char* ap = &firstCharA[0]) fixed (char* bp = &firstCharB[0])
2、(*(int*)a->获取的数据是两个char值（低位ASCII*65536+高位ASCII）[低位在前，高位在后]。 [char两个字节,范围U+0000到U+FFFF]
3、(*(char*)a->获取的数据是一个char值[见上面测试例子]

```

4.1.7、测试EqualsHelper(String strA, String strB)函数

```c#
            {
                string a = "abcd";
                string b = "abcd";
                Console.WriteLine(EqualsHelper(a,b));//返回true
            }

            {
                string a = "Test";
                string b = string.Copy(a);
                Console.WriteLine(EqualsHelper(a, b));//返回true
            }

            {
                string a = "Test";
                string b = (string)a.Clone();
                Console.WriteLine(EqualsHelper(a, b));//返回true
            }

            {
                char[] ch = new char[] { 'a', 'A', '@' };
                string a = "aA@";
                string b = new string(ch);
                Console.WriteLine(EqualsHelper(a, b));//返回true
            }

```

4.1.8、结论：string类型 a == b、string.Equals(a, b)、a.Equals(b)、a.Equals((object)b)，如果 a 的值与 b 的值相同，则为 true；否则为 false。

4.2、类class

4.2.1、C#语言规范5.0中文版中的引用类型相等运算符介绍
![5](/images/c%23Equal函数and运算符'%3D%3D'/5.PNG)
![6](/images/c%23Equal函数and运算符'%3D%3D'/6.PNG)

4.2.2、Object内部的Equals函数

```c#
		Object
		{
			public virtual bool Equals(Object obj)
		    {
		        return RuntimeHelpers.Equals(this, obj);//无法查到详细代码
		    }
		 
		    public static bool Equals(Object objA, Object objB) 
		    {
		        if (objA==objB) {
		            return true;
		        }
		        if (objA==null || objB==null) {
		            return false;
		        }
		        return objA.Equals(objB);
		    }
		 
		    [ReliabilityContract(Consistency.WillNotCorruptState, Cer.Success)]
		    [System.Runtime.Versioning.NonVersionable]
		    public static bool ReferenceEquals (Object objA, Object objB) {
		        return objA == objB;
		    }
		} 

```

4.2.3、类(不重写Equal函数 and 运算符'==')

```c#
    public class RefPoint
    {
        public double x;
        public double y;
        public double z;

        public RefPoint(double X, double Y, double Z)
        {
            this.x = X;
            this.y = Y;
            this.z = Z;
        }
    }
    
            RefPoint p1 = new RefPoint(4, 5, 6);
            RefPoint p2 = p1;
            Console.WriteLine(p1.Equals(p2));//返回true
            Console.WriteLine(object.Equals(p1, p2));//返回true
            Console.WriteLine(object.ReferenceEquals(p1, p2));//返回true
            Console.WriteLine(p1 == p2);//返回true

            p2 = new RefPoint(4, 5, 6);//虽然值一样，但是引用对象不一样
            Console.WriteLine(p1.Equals(p2));//返回false
            Console.WriteLine(object.Equals(p1, p2));//返回false
            Console.WriteLine(object.ReferenceEquals(p1, p2));//返回false
            Console.WriteLine(p1 == p2);//返回false

```

4.2.4、类(重写Equal函数 and 运算符'==')

```c#
    public class RefPoint
    {
        public double x;
        public double y;
        public double z;

        public RefPoint(double X, double Y, double Z)
        {
            this.x = X;
            this.y = Y;
            this.z = Z;
        }

        public override bool Equals(Object obj)
        {
            if (!(obj is RefPoint))
            {
                return false;
            }

            if (((RefPoint)obj).x == this.x)
            {
                return true;
            }

            return false;
        }

        public bool Equals(RefPoint obj)
        {
            if (obj.x == this.x)
            {
                return true;
            }

            return false;
        }

        public static bool operator ==(RefPoint left, RefPoint right)
        {
            return left.x == right.x;
        }

        public static bool operator !=(RefPoint left, RefPoint right)
        {
            return left.x != right.x;
        }
    }

            RefPoint p1 = new RefPoint(4, 5, 6);
            RefPoint p2 = p1;
            Console.WriteLine(p1.Equals(p2));//返回true
            Console.WriteLine(object.Equals(p1, p2));//返回true
            Console.WriteLine(object.ReferenceEquals(p1, p2));//返回true
            Console.WriteLine(p1 == p2);//返回true

            p2 = new RefPoint(4, 50, 60);
            Console.WriteLine(p1.Equals(p2));//返回true
            Console.WriteLine(object.Equals(p1, p2));//返回true
            Console.WriteLine(object.ReferenceEquals(p1, p2));//返回false
            Console.WriteLine(p1 == p2);//返回true

```

4.2.5、ReferenceEquals (Object objA, Object objB)返回objA == objB。如果objA、 objB引用同一个对象(只判断是否引用同一个对象，即使我们自行重载了'=='运算符，也没用)，返回true；否则，返回false。

5、总结

先介绍简单值类型，再到结构体，字符串，类。把每个类型Equal和'=='用法做个总结，加深自己记忆的同时，也希望能帮助到你。另：本文只代表本人观点，如果有误，还望告知。

6、参考
6.1、[C#类型基础](http://www.cnblogs.com/JimmyZhang/archive/2008/01/31/1059383.html)
6.2、 C#语言规范5.0中文版
6.3、[Reference Source ](http://referencesource.microsoft.com/#mscorlib)
