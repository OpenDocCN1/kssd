
`DalvikVM64.java`中修改`CallStaticBooleanMethodV`的hook代码，用于支持函数直接初始化调用参数后直接修改`pc`来`call`，通过`R0`来区分判断是否是这种方式调用。

```java
public void initArgs(Emulator<?> emulator, Object...args) {
    if (objectType == null) {
        throw new IllegalStateException("objectType is null");
    }
    initArgs(emulator, vm, this, args);
}
public static Arguments initArgs(Emulator<?> emulator, VM vm, DvmObject<?> thisObj, Object...args) {
    List<Object> list = new ArrayList<>(10);
    list.add(vm.getJNIEnv());
    list.add(thisObj.hashCode());
    if (args != null) {
        for (Object arg : args) {
            if (arg instanceof Boolean) {
                list.add((Boolean) arg ? VM.JNI_TRUE : VM.JNI_FALSE);
                continue;
            } else if (arg instanceof Hashable) {
                list.add(arg.hashCode());
                // dvm object
                if (arg instanceof DvmObject) {
                    vm.addLocalObject((DvmObject<?>) arg);
                }
                continue;
            } else if (arg instanceof String) {
                StringObject str = new StringObject(vm, (String) arg);
                list.add(str.hashCode());
                vm.addLocalObject(str);
                continue;
            } else if (arg instanceof byte[]) {
                ByteArray array = new ByteArray(vm, (byte[]) arg);
                list.add(array.hashCode());
                vm.addLocalObject(array);
                continue;
            }
            list.add(arg);
        }
    }
    args = list.toArray();
    List<Number> numList = new ArrayList<>(args.length);
    for (Object arg : args) {
        if (arg instanceof String) {
            numList.add(new StringNumber((String) arg));
        } else if (arg instanceof byte[]) {
            numList.add(new ByteArrayNumber((byte[]) arg));
        } else if (arg instanceof UnidbgPointer) {
            UnidbgPointer pointer = (UnidbgPointer) arg;
            numList.add(new PointerNumber(pointer));
        } else if (arg instanceof UnidbgStructure) {
            UnidbgStructure structure = (UnidbgStructure) arg;
            numList.add(new PointerNumber((UnidbgPointer) structure.getPointer()));
        } else if (arg instanceof Number) {
            numList.add((Number) arg);
        } else if (arg == null) {
            numList.add(new PointerNumber(null));
            // null
        } else {
            throw new IllegalStateException("Unsupported arg: " + arg);
        }
    }
    return ARM.initArgs(emulator, true, numList.toArray(new Number[0]));
}
```

```java
Pointer _CallStaticBooleanMethodV = svcMemory.registerSvc(new Arm64Svc() {
    @Override
    public long handle(Emulator<?> emulator) {
        UnidbgPointer clazz = UnidbgPointer.register(emulator, Arm64Const.UC_ARM64_REG_X1);
        UnidbgPointer jmethodID = UnidbgPointer.register(emulator, Arm64Const.UC_ARM64_REG_X2);
        UnidbgPointer va_list = UnidbgPointer.register(emulator, Arm64Const.UC_ARM64_REG_X3);
        if (log.isDebugEnabled()) {
            log.debug("CallStaticBooleanMethodV clazz=" + clazz + ", jmethodID=" + jmethodID + ", va_list=" + va_list);
        }
        DvmClass dvmClass = classMap.get(clazz.toIntPeer());
        DvmMethod dvmMethod = dvmClass == null ? null : dvmClass.getStaticMethod(jmethodID.toIntPeer());
        if (dvmMethod == null) {
            throw new BackendException();
        } else {
            long oriX0 = emulator.getBackend().reg_read(Arm64Const.UC_ARM64_REG_X0).longValue();
            VaList vaList = new VaList64(emulator, DalvikVM64.this, va_list, dvmMethod);
            long ret = dvmMethod.callStaticBooleanMethodV(vaList);
            long newX0 = emulator.getBackend().reg_read(Arm64Const.UC_ARM64_REG_X0).longValue();
            if (oriX0 != newX0) {
                ret = newX0 - 1;
            }
            if (verbose) {
                System.out.printf("JNIEnv->CallStaticBooleanMethodV(%s, %s(%s) => %s) was called from %s%n", dvmClass, dvmMethod.methodName, vaList.formatArgs(), ret == JNI_TRUE, UnidbgPointer.register(emulator, Arm64Const.UC_ARM64_REG_LR));
            }
            return ret;
        }
    }
});
```

`MainActivity.java` 主要的模拟调用代码
