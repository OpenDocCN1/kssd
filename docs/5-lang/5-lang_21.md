
# .net 2.0 下利用动态 Assembly 与 Reflect 机制向 DHTML 暴露 .net framework

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-190.htm`](https://www.pediy.com/kssd/pediy08/pediy8-190.htm)

**【文章标题】**: .net 2.0 下利用动态 Assembly 与 Reflect 机制向 DHTML 暴露 .net 应用里的对象与 .net framework  
**【文章作者】**: dreaman  
**【作者邮箱】**: `dreaman_163@163.com`  
**【作者主页】**: `http://dreaman.haolinju.net`  
**【作者声明】**: 主要用于为 `WebBrowser` 控件提供扩展对象，不过涉及动态代码编写，也许对 crack 有用。

---

### 【详细过程】

`.net` 2.0 提供的 `WebBrowser` 控件对 `IE` 的封装十分酷,特别是其 `ObjectForScripting` 特性,只要将一个 `.net` 里的对象实例赋给它,就可以在 `DHTML` 中用 `window.external` 访问该 `.net` 对象了(有一个限制是该 `.net` 对象必须使用 `ComVisible` 属性修饰成 COM 可见的),如果用 `IE` 作界面,`DHTML` 可以认为是 UI 与控制器,`.NET` 中则实现对象模型,如果可以在 `DHTML` 中自由的访问 `.NET` 里的对象,那会让人很舒服.

我想到的方法是在 `external` 上提供一个 `CreateObject` 方法,直接由脚本来创建 `.net` 里的对象实例,但是有一个问题,`.net` 里有许多对象并不是 COM 可见的,这样的对象实例无法返回给 `DHTML`,所以必须对这样的对象作些处理.

对非 COM 可见的对象,要让 `DHTML` 来访问,必须作一个代理,代理对象设成 COM 可见的,然后代理对象具有原对象一样的访问接口.因为目标客户是 `javascript` 脚本,它访问对象是通过 `IDispatch[Ex]` 接口来实现的,所以我们的代理对象实际上并不需要与原对象有完全一致的接口,这样,有三种可能的方式:

1. 直接修原 `.NET` 对象，让它变成 COM 可见的；
2. 在 `.NET` 中实现一个对象，它实现 `IDispatch` 接口，实现的功能是访问原 `.net` 对象的属性与方法；
3. 利用动态装配与反射，动态的生成一个代理类，接口与原 `.net` 类相同，并设置为 COM 可见的。

方法 1 我不知道如何做，因为 `.net` 框架没有提供对现有类的修改的方式，如果是 cracker 的方式，我想可能会比较复杂并且不通用（更重要的是，我不知道怎么弄，谁要是知道，请告诉我，谢了）。

方法 2 理论上是绝对可行的，但是我打算用 `C#` 来写，也就是说企图用托管代码实现 `IDispatch` 接口，实验失败，`MSDN` 上好象是说,`CLR` 总是会替 `.net` 对象实现 `IUnknown` 与 `IDispatch` 接口；如果是用 `C++`，我想 `IDispatch` 实现成非托管的，然后其实现功能利用 `C++` 互操作调 `.NET` 的相关实现，然后将非托管 `IDispatch` 实例返回并使用自己的定制 `Marshal` 应该是可行的（没有试验过，只是想法）。

现在只剩下方法 3 了，我试验了这种方法，可行。

不多说了，直接看代码（试验性的，其中的定制 `Marshal` 是试验用的，现在的实现与 `.NET` 默认的 `Marshal` 是一样的，所以是可以去掉的）

```csharp
//定制 Marshal，实现与.net 运行时作的工作一样，可以去掉
public class MyMarshal : ICustomMarshaler
{
    #region ICustomMarshaler 成员
    public void CleanUpManagedData(object ManagedObj)
    {}
    public void CleanUpNativeData(IntPtr pNativeData)
    {}
    public int GetNativeDataSize()
    {
        return -1;
    }
    public IntPtr MarshalManagedToNative(object o)
    {
        IntPtr i = Marshal.GetIDispatchForObject(o);
        return i;
    }
    public object MarshalNativeToManaged(IntPtr pNativeData)
    {
        throw new Exception("The method or operation is not implemented.");
    }
    #endregion
    static public ICustomMarshaler GetInstance(string pstrCookie)
    {
        return ins;
    }
    static private MyMarshal ins = new MyMarshal();
}

//扩展对象的实现   
[ComVisible(true)]
public class External
{
    //判断一个.net 对象实例是否是 COM 可见的，如果不可见则实时生成一个 COM 可见的代理类，并返回代理类的实例。
    [method: ComVisible(false)]
    static private object WrapAsComVisible(Type t, object o)
    {
        Attribute[] attrs = Attribute.GetCustomAttributes(t);
        foreach (Attribute a in attrs)
        {
            if (a is ComVisibleAttribute)
            {
                ComVisibleAttribute cva = a as ComVisibleAttribute;
                if (cva.Value)
                    return o;
            }
        }

        //现在开始生成 COM 可见的代理类
        AssemblyName an = new AssemblyName("__" + t.Namespace + "_" + t.Name + "_" + t.GetHashCode());
        AssemblyBuilder ab = AppDomain.CurrentDomain.DefineDynamicAssembly(an, AssemblyBuilderAccess.Run);
        ModuleBuilder mob = ab.DefineDynamicModule("_" + t.Name + "_" + t.GetHashCode());
        TypeBuilder tb = mob.DefineType(t.Name + "_" + t.GetHashCode(), TypeAttributes.Public);
        Type[] cts = new Type[] { typeof(bool) };
        ConstructorInfo ci = typeof(ComVisibleAttribute).GetConstructor(cts);
        CustomAttributeBuilder cab = new CustomAttributeBuilder(ci, new object[] { true });
        tb.SetCustomAttribute(cab);
        tb.DefineDefaultConstructor(MethodAttributes.Public);
        FieldBuilder innerFieldInfo = tb.DefineField("innerObj", t, FieldAttributes.Public);
        MemberInfo[] memberInfos = t.GetMembers();
        foreach (MemberInfo memberInfo in memberInfos)
        {
            switch (memberInfo.MemberType)
            {
                case MemberTypes.Field:
                    {
                        FieldInfo fi = memberInfo as FieldInfo;
                        PropertyBuilder pb = tb.DefineProperty(fi.Name, System.Reflection.PropertyAttributes.None, fi.FieldType, null);
                        if (true)
                        {
                            MethodBuilder getMB = tb.DefineMethod("get_" + fi.Name, MethodAttributes.Public | MethodAttributes.SpecialName, fi.FieldType, null);
                            ILGenerator ilg = getMB.GetILGenerator();
                            ilg.Emit(OpCodes.Ldarg_0);
                            ilg.Emit(OpCodes.Ldfld, innerFieldInfo);
                            ilg.Emit(OpCodes.Ldfld, fi);
                            ilg.Emit(OpCodes.Ret);
                            pb.SetGetMethod(getMB);
                        }
                        if (true)
                        {
                            Type[] tis = new Type[] { fi.FieldType };
                            MethodBuilder setMB = tb.DefineMethod("set_" + fi.Name, MethodAttributes.Public | MethodAttributes.SpecialName, typeof(void), tis);
                            ILGenerator ilg = setMB.GetILGenerator();
                            ilg.Emit(OpCodes.Ldarg_0);
                            ilg.Emit(OpCodes.Ldfld, innerFieldInfo);
                            ilg.Emit(OpCodes.Ldarg_1);
                            ilg.Emit(OpCodes.Stfld, fi);
                            ilg.Emit(OpCodes.Ret);
                            pb.SetSetMethod(setMB);
                        }
                    }
                    break;
                case MemberTypes.Property:
                    {
                        PropertyInfo pi = memberInfo as PropertyInfo;
                        ParameterInfo[] paras = pi.GetIndexParameters();
                        Type[] tis0 = new Type[paras.Length];
                        for (int i = 0; i < tis0.Length; i++)
                        {
                            tis0[i] = paras[i].ParameterType;
                        }
                        PropertyBuilder pb = tb.DefineProperty(pi.Name, System.Reflection.PropertyAttributes.None, pi.PropertyType, tis0);
                        if (pi.CanRead)
                        {
                            if (paras.Length > 0)
                            {
                                Type[] tis = new Type[paras.Length];
                                for (int i = 0; i < tis.Length; i++)
                                {
                                    tis[i] = paras[i].ParameterType;
                                }
                                MethodBuilder getMB = tb.DefineMethod("get_Item", MethodAttributes.Public | MethodAttributes.SpecialName, pi.PropertyType, tis);
                                ILGenerator ilg = getMB.GetILGenerator();
                                ilg.Emit(OpCodes.Ldarg_0);
```



```
            ilg.Emit(OpCodes.Ldfld, innerFieldInfo);  

            for (int i = 0; i < tis.Length; i++)  

            {  

                switch (i)  

                {  

                    case 0:  

                        {  

                            ilg.Emit(OpCodes.Ldarg_1);  

                        }  

                        break;  

                    case 1:  

                        {  

                            ilg.Emit(OpCodes.Ldarg_2);  

                        }  

                        break;  

                    case 2:  

                        {  

                            ilg.Emit(OpCodes.Ldarg_3);  

                        }  

                        break;  

                    default:  

                        {  

                            ilg.Emit(OpCodes.Ldarg_S, i + 1);  

                        }  

                        break;  

                }  

            }  

            ilg.EmitCall(OpCodes.Call, pi.GetGetMethod(), tis);  

            ilg.Emit(OpCodes.Ret);  

            pb.SetGetMethod(getMB);  

        }  

        else  

        {  

            MethodBuilder getMB = tb.DefineMethod("get_" + pi.Name, MethodAttributes.Public | MethodAttributes.SpecialName, pi.PropertyType, null);  

            ILGenerator ilg = getMB.GetILGenerator();  

            ilg.Emit(OpCodes.Ldarg_0);  

            ilg.Emit(OpCodes.Ldfld, innerFieldInfo);  

            ilg.EmitCall(OpCodes.Call, pi.GetGetMethod(), null);  

            ilg.Emit(OpCodes.Ret);  

            pb.SetGetMethod(getMB);  

        }  

    }  

    if (pi.CanWrite)  

    {  

        if (paras.Length > 0)  

        {  

            Type[] tis = new Type[paras.Length + 1];  

            for (int i = 0; i < tis.Length; i++)  

            {  

                if (i < paras.Length)  

                    tis[i] = paras[i].ParameterType;  

                else  

                    tis[i] = pi.PropertyType;  

            }  

            MethodBuilder setMB = tb.DefineMethod("set_Item", MethodAttributes.Public | MethodAttributes.SpecialName, typeof(void), tis);  

            ILGenerator ilg = setMB.GetILGenerator();  

            ilg.Emit(OpCodes.Ldarg_0);  

            ilg.Emit(OpCodes.Ldfld, innerFieldInfo);  

            for (int i = 0; i < tis.Length; i++)  

            {  

                switch (i)  

                {  

                    case 0:  

                        {  

                            ilg.Emit(OpCodes.Ldarg_1);  

                        }  

                        break;  

                    case 1:  

                        {  

                            ilg.Emit(OpCodes.Ldarg_2);  

                        }  

                        break;  

                    case 2:  

                        {  

                            ilg.Emit(OpCodes.Ldarg_3);  

                        }  

                        break;  

                    default:  

                        {  

                            ilg.Emit(OpCodes.Ldarg_S, i + 1);  

                        }  

                        break;  

                }  

            }  

            ilg.EmitCall(OpCodes.Call, pi.GetSetMethod(), tis);  

            ilg.Emit(OpCodes.Ret);  

            pb.SetSetMethod(setMB);  

        }  

        else  

        {  

            Type[] tis = new Type[] { pi.PropertyType };  

            MethodBuilder setMB = tb.DefineMethod("set_" + pi.Name, MethodAttributes.Public | MethodAttributes.SpecialName, typeof(void), tis);  

            ILGenerator ilg = setMB.GetILGenerator();  

            ilg.Emit(OpCodes.Ldarg_0);  

            ilg.Emit(OpCodes.Ldfld, innerFieldInfo);  

            ilg.Emit(OpCodes.Ldarg_1);  

            ilg.EmitCall(OpCodes.Call, pi.GetSetMethod(), tis);  

            ilg.Emit(OpCodes.Ret);  

            pb.SetSetMethod(setMB);  

        }  

    }  

}  

break;  

case MemberTypes.Method:  

    {  

        MethodInfo mi = memberInfo as MethodInfo;  

        MethodBuilder mb = tb.DefineMethod(mi.Name, MethodAttributes.Public);  

        ParameterInfo[] paras = mi.GetParameters();  

        Type[] tis = new Type[paras.Length];  

        for (int i = 0; i < tis.Length; i++)  

        {  

            tis[i] = paras[i].ParameterType;  

        }  

        mb.SetReturnType(mi.ReturnType);  

        mb.SetParameters(tis);  

        ILGenerator ilg = mb.GetILGenerator();  

        ilg.Emit(OpCodes.Ldarg_0);  

        ilg.Emit(OpCodes.Ldfld, innerFieldInfo);  

        for (int i = 0; i < tis.Length; i++)  

        {  

            switch (i)  

            {  

                case 0:  

                    {  

                        ilg.Emit(OpCodes.Ldarg_1);
```



```csharp
                    }  
                    break;  
                case 1:  
                    {  
                        ilg.Emit(OpCodes.Ldarg_2);  
                    }  
                    break;  
                case 2:  
                    {  
                        ilg.Emit(OpCodes.Ldarg_3);  
                    }  
                    break;  
                default:  
                    {  
                        ilg.Emit(OpCodes.Ldarg_S, i + 1);  
                    }  
                    break;  
                }  
            }  
            ilg.EmitCall(OpCodes.Call, t.GetMethod(mi.Name), tis);  
            ilg.Emit(OpCodes.Ret);  
        }  
        break;  
    }  
}  

Type tt = tb.CreateType();  

//创建代理类实例并初始化指向原.net 类实例的指针。  

    object oo = Activator.CreateInstance(tt);  

    tt.InvokeMember("innerObj", BindingFlags.SetField, null, oo, new object[] { o });  

    return oo;  
}  

[method: ComVisible(false)]  

static private object CreateObjectFromFileX(string aname, string tname, params object[] args)  
{  
    Assembly a = Assembly.LoadFile(aname);  

    Type t = a.GetType(tname);  

    object o=Activator.CreateInstance(t, args);  

    return WrapAsComVisible(t, o);  
}  

[method: ComVisible(false)]  

static private object CreateObjectX(string tname, params object[] args)  
{  
    Type t = Type.GetType(tname);  

    object o = Activator.CreateInstance(t, args);  

    return WrapAsComVisible(t, o);     
}  

    /*  
        后续的方法是前面两个方法的特定参数数的版本，因为客户是用访问 COM 对象的方式访问 External，我们  
        为其提供参数固定的访问形式。  
    */  

    [return: MarshalAs(UnmanagedType.CustomMarshaler, MarshalTypeRef = typeof(MyMarshal))]  

    public object CreateObjectFromFile0(string aname, string tname)  
    {  
        return CreateObjectFromFileX(aname, tname);  
    }  

    [return: MarshalAs(UnmanagedType.CustomMarshaler, MarshalTypeRef = typeof(MyMarshal))]  

    public object CreateObject0(string tname)  
    {  
        return CreateObjectX(tname);  
    }  

    [return: MarshalAs(UnmanagedType.CustomMarshaler, MarshalTypeRef = typeof(MyMarshal))]  

    public object CreateObjectFromFile1(string aname, string tname, object arg1)  
    {  
        return CreateObjectFromFileX(aname, tname, arg1);  
    }  

    [return: MarshalAs(UnmanagedType.CustomMarshaler, MarshalTypeRef = typeof(MyMarshal))]  

    public object CreateObject1(string tname, object arg1)  
    {  
        return CreateObjectX(tname, arg1);  
    }  

    [return: MarshalAs(UnmanagedType.CustomMarshaler, MarshalTypeRef = typeof(MyMarshal))]  

    public object CreateObjectFromFile2(string aname, string tname, object arg1, object arg2)  
    {  
        return CreateObjectFromFileX(aname, tname, arg1, arg2);  
    }  

    [return: MarshalAs(UnmanagedType.CustomMarshaler, MarshalTypeRef = typeof(MyMarshal))]  

    public object CreateObject2(string tname, object arg1, object arg2)  
    {  
        return CreateObjectX(tname, arg1, arg2);  
    }  

    [return: MarshalAs(UnmanagedType.CustomMarshaler, MarshalTypeRef = typeof(MyMarshal))]  

    public object CreateObjectFromFile3(string aname, string tname, object arg1, object arg2, object arg3)  
    {  
        return CreateObjectFromFileX(aname, tname, arg1, arg2, arg3);  
    }  

    [return: MarshalAs(UnmanagedType.CustomMarshaler, MarshalTypeRef = typeof(MyMarshal))]  

    public object CreateObject3(string tname, object arg1, object arg2, object arg3)  
    {  
        return CreateObjectX(tname, arg1, arg2, arg3);  
    }  

    [return: MarshalAs(UnmanagedType.CustomMarshaler, MarshalTypeRef = typeof(MyMarshal))]  

    public object CreateObjectFromFile4(string aname, string tname, object arg1, object arg2, object arg3, object arg4)  
    {  
        return CreateObjectFromFileX(aname, tname, arg1, arg2, arg3, arg4);  
    }  

    [return: MarshalAs(UnmanagedType.CustomMarshaler, MarshalTypeRef = typeof(MyMarshal))]  

    public object CreateObject4(string tname, object arg1, object arg2, object arg3, object arg4)  
    {  
        return CreateObjectX(tname, arg1, arg2, arg3, arg4);  
    }  

    [return: MarshalAs(UnmanagedType.CustomMarshaler, MarshalTypeRef = typeof(MyMarshal))]  

    public object CreateObjectFromFile5(string aname, string tname, object arg1, object arg2, object arg3, object arg4, object arg5)  
    {  
        return CreateObjectFromFileX(aname, tname, arg1, arg2, arg3, arg4, arg5);  
    }  

    [return: MarshalAs(UnmanagedType.CustomMarshaler, MarshalTypeRef = typeof(MyMarshal))]  

    public object CreateObject5(string tname, object arg1, object arg2, object arg3, object arg4, object arg5)  
    {  
        return CreateObjectX(tname, arg1, arg2, arg3, arg4, arg5);  
    }  

    [return: MarshalAs(UnmanagedType.CustomMarshaler, MarshalTypeRef = typeof(MyMarshal))]  

    public object CreateObjectFromFile6(string aname, string tname, object arg1, object arg2, object arg3, object arg4, object arg5, object arg6)  
    {  
        return CreateObjectFromFileX(aname, tname, arg1, arg2, arg3, arg4, arg5, arg6);  
    }  

    [return: MarshalAs(UnmanagedType.CustomMarshaler, MarshalTypeRef = typeof(MyMarshal))]  

    public object CreateObject6(string tname, object arg1, object arg2, object arg3, object arg4, object arg5, object arg6)  
    {  
        return CreateObjectX(tname, arg1, arg2, arg3, arg4, arg5, arg6);  
    }  

    [return: MarshalAs(UnmanagedType.CustomMarshaler, MarshalTypeRef = typeof(MyMarshal))]  

    public object CreateObjectFromFile7(string aname, string tname, object arg1, object arg2, object arg3, object arg4, object arg5, object arg6, object arg7)  
    {  
        return CreateObjectFromFileX(aname, tname, arg1, arg2, arg3, arg4, arg5, arg6, arg7);  
    }  

    [return: MarshalAs(UnmanagedType.CustomMarshaler, MarshalTypeRef = typeof(MyMarshal))]  

    public object CreateObject7(string tname, object arg1, object arg2, object arg3, object arg4, object arg5, object arg6, object arg7)  
    {  
        return CreateObjectX(tname, arg1, arg2, arg3, arg4, arg5, arg6, arg7);  
    }  

    [return: MarshalAs(UnmanagedType.CustomMarshaler, MarshalTypeRef = typeof(MyMarshal))]  

    public object CreateObjectFromFile8(string aname, string tname, object arg1, object arg2, object arg3, object arg4, object arg5, object arg6, object arg7, object arg8)  
    {  
        return CreateObjectFromFileX(aname, tname, arg1, arg2, arg3, arg4, arg5, arg6, arg7, arg8);  
    }  

    [return: MarshalAs(UnmanagedType.CustomMarshaler, MarshalTypeRef = typeof(MyMarshal))]  
```



```
public object CreateObject8(string tname, object arg1, object arg2, object arg3, object arg4, object arg5, object arg6, object arg7, object arg8)  
{  
    return CreateObjectX(tname, arg1, arg2, arg3, arg4, arg5, arg6, arg7, arg8);  
}  

}  
```

用法很简单，实例化一个`External`并将它赋给`WebBrowser`控件实例的`ObjectForScripting`，然后在`DHTML`里用`window.external.CreateObject`系列方法来创建`.net`里的对象（包括当前`EXE`中实现的与`.net`框架库及`dll`的），可以看出，相当于向`DHTML`暴露了整个`.net framework`，呵呵。

代理的实现在`WrapAsComVisible`中。

---

### 【经验总结】

`.net`中的`reflect`与动态装配使得我们可以在运行时构造并执行代码（相当于`win32`程序里在内存中写入并执行指令），不过我不知道如何修改已经存在的代码与数据，如果可以，则目前`win32`代码钩子所能做的一切`.NET`里都可以做到（如果哪位知道怎么在运行时修改`.net`现行的`IL`与元数据，请告诉我）。

---

2006 年 05 月 22 日 12:24:28

`4dd466d0e1e0cc123afbe504da2462ca`
