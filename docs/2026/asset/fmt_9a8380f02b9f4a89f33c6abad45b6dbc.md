
```
; ModuleID = 'test.bc'
source\_filename = "test.cpp"
target datalayout = "e-m:w-p270:32:32-p271:32:32-p272:64:64-i64:64-f80:128-n8:16:32:64-S128"
target triple = "x86\_64-w64-windows-gnu"
$\_Z6printfPKcz = comdat any
@.str = private unnamed\_addr global [8 x i8] c"&e;;;&e\01", align 1
@.str.1 = private unnamed\_addr global [8 x i8] c"Ifmmp\222\01", align 1
@.str.2 = private unnamed\_addr global [8 x i8] c"Ifmmp\223\01", align 1
@.str.3 = private unnamed\_addr global [8 x i8] c"Ifmmp\224\01", align 1
@.str.4 = private unnamed\_addr global [8 x i8] c"Ifmmp\225\01", align 1
@.str.5 = private unnamed\_addr global [9 x i8] c"Ifmmp\2221\01", align 1
@.str.6 = private unnamed\_addr global [12 x i8] c"ifmmp!Qbtt\22\01", align 1
@reltable.main = private unnamed\_addr constant [5 x i32] [i32 trunc (i64 sub (i64 ptrtoint (ptr @.str.1 to i64), i64 ptrtoint (ptr @reltable.main to i64)) to i32), i32 trunc (i64 sub (i64 ptrtoint (ptr @.str.4 to i64), i64 ptrtoint (ptr @reltable.main to i64)) to i32), i32 trunc (i64 sub (i64 ptrtoint (ptr @.str.2 to i64), i64 ptrtoint (ptr @reltable.main to i64)) to i32), i32 trunc (i64 sub (i64 ptrtoint (ptr @.str.4 to i64), i64 ptrtoint (ptr @reltable.main to i64)) to i32), i32 trunc (i64 sub (i64 ptrtoint (ptr @.str.3 to i64), i64 ptrtoint (ptr @reltable.main to i64)) to i32)], align 4
; Function Attrs: mustprogress nofree nounwind uwtable
define dso\_local void @\_Z4fun1PKc(ptr nocapture noundef readonly %0) local\_unnamed\_addr #0 {
%2 = tail call i32 @puts(ptr noundef nonnull dereferenceable(1) %0)
ret void
}
; Function Attrs: nofree nounwind
declare dso\_local noundef i32 @puts(ptr nocapture noundef readonly) local\_unnamed\_addr #1
; Function Attrs: mustprogress uwtable
define dso\_local void @\_Z4fun2ii(i32 noundef %0, i32 noundef %1) local\_unnamed\_addr #2 {
%3 = tail call noundef i32 (ptr, ...) @\_Z6printfPKcz(ptr noundef @.str, i32 noundef %0, i32 noundef %1)
ret void
}
; Function Attrs: inlinehint mustprogress uwtable
define linkonce\_odr dso\_local noundef i32 @\_Z6printfPKcz(ptr noundef nonnull %0, ...) local\_unnamed\_addr #3 comdat {
%2 = alloca ptr, align 8
call void @llvm.lifetime.start.p0(i64 8, ptr nonnull %2) #10
call void @llvm.va\_start(ptr nonnull %2)
%3 = call ptr @\_\_acrt\_iob\_func(i32 noundef 1)
%4 = load ptr, ptr %2, align 8, !tbaa !4
%5 = call i32 @\_\_mingw\_vfprintf(ptr noundef %3, ptr noundef nonnull %0, ptr noundef %4) #10
call void @llvm.va\_end(ptr %2)
call void @llvm.lifetime.end.p0(i64 8, ptr nonnull %2) #10
ret i32 %5
}
; Function Attrs: mustprogress norecurse uwtable
define dso\_local noundef i32 @main(i32 noundef %0, ptr nocapture noundef readnone %1) local\_unnamed\_addr #4 {
DecodeStub:
call void @decode\_stub()
br label %2
2: ; preds = %DecodeStub
%3 = add i32 %0, -5
%4 = icmp ult i32 %3, 5
br i1 %4, label %5, label %9
5: ; preds = %2
%6 = sext i32 %3 to i64
%7 = shl i64 %6, 2
%8 = call ptr @llvm.load.relative.i64(ptr @reltable.main, i64 %7)
br label %9
9: ; preds = %5, %2
%10 = phi ptr [ %8, %5 ], [ @.str.4, %2 ]
%11 = tail call i32 @puts(ptr noundef nonnull dereferenceable(1) %10)
%12 = icmp slt i32 %0, 5
br i1 %12, label %13, label %15
13: ; preds = %9
%14 = tail call i32 @puts(ptr noundef nonnull dereferenceable(1) @.str.5)
br label %15
15: ; preds = %13, %9
%16 = tail call i32 @puts(ptr noundef nonnull dereferenceable(1) @.str.6)
%17 = tail call noundef i32 (ptr, ...) @\_Z6printfPKcz(ptr noundef @.str, i32 noundef 5, i32 noundef 6)
ret i32 0
}
; Function Attrs: nocallback nofree nosync nounwind willreturn memory(argmem: readwrite)
declare void @llvm.lifetime.start.p0(i64 immarg, ptr nocapture) #5
; Function Attrs: nocallback nofree nosync nounwind willreturn
declare void @llvm.va\_start(ptr) #6
; Function Attrs: nounwind
declare dso\_local i32 @\_\_mingw\_vfprintf(ptr noundef, ptr noundef, ptr noundef) local\_unnamed\_addr #7
declare dllimport ptr @\_\_acrt\_iob\_func(i32 noundef) local\_unnamed\_addr #8
; Function Attrs: nocallback nofree nosync nounwind willreturn
declare void @llvm.va\_end(ptr) #6
; Function Attrs: nocallback nofree nosync nounwind willreturn memory(argmem: readwrite)
declare void @llvm.lifetime.end.p0(i64 immarg, ptr nocapture) #5
; Function Attrs: nocallback nofree nosync nounwind willreturn memory(argmem: read)
declare ptr @llvm.load.relative.i64(ptr, i64) #9
define void @decode(ptr addrspace(8) %0, i32 %1) {
%var3 = icmp ne ptr addrspace(8) %0, null
%3 = icmp sgt i32 %1, 0
%var5 = and i1 %3, %var3
br i1 %var5, label %bb6, label %bb14
bb6: ; preds = %bb6, %2
%var7 = phi ptr addrspace(8) [ %0, %2 ], [ %var10, %bb6 ]
%var8 = phi i32 [ %1, %2 ], [ %var9, %bb6 ]
%var9 = add nsw i32 %var8, -1
%var10 = getelementptr i8, ptr addrspace(8) %var7, i8 1
%var2 = load i8, ptr addrspace(8) %var7, align 1
%var12 = add i8 %var2, -1
store i8 %var12, ptr addrspace(8) %var7, align 1
%var13 = icmp sgt i32 %var8, 0
br i1 %var13, label %bb6, label %bb14
bb14: ; preds = %bb6, %2
ret void
}
define void @decode\_stub() {
entry:
call void @decode(ptr addrspace(8) addrspacecast (ptr @.str to ptr addrspace(8)), i32 7)
call void @decode(ptr addrspace(8) addrspacecast (ptr @.str.1 to ptr addrspace(8)), i32 7)
call void @decode(ptr addrspace(8) addrspacecast (ptr @.str.2 to ptr addrspace(8)), i32 7)
call void @decode(ptr addrspace(8) addrspacecast (ptr @.str.3 to ptr addrspace(8)), i32 7)
call void @decode(ptr addrspace(8) addrspacecast (ptr @.str.4 to ptr addrspace(8)), i32 7)
call void @decode(ptr addrspace(8) addrspacecast (ptr @.str.5 to ptr addrspace(8)), i32 8)
call void @decode(ptr addrspace(8) addrspacecast (ptr @.str.6 to ptr addrspace(8)), i32 11)
ret void
}
attributes #0 = { mustprogress nofree nounwind uwtable "min-legal-vector-width"="0" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
attributes #1 = { nofree nounwind "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
attributes #2 = { mustprogress uwtable "min-legal-vector-width"="0" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
attributes #3 = { inlinehint mustprogress uwtable "min-legal-vector-width"="0" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
attributes #4 = { mustprogress norecurse uwtable "min-legal-vector-width"="0" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
attributes #5 = { nocallback nofree nosync nounwind willreturn memory(argmem: readwrite) }
attributes #6 = { nocallback nofree nosync nounwind willreturn }
attributes #7 = { nounwind "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
attributes #8 = { "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
attributes #9 = { nocallback nofree nosync nounwind willreturn memory(argmem: read) }
attributes #10 = { nounwind }
!llvm.module.flags = !{!0, !1, !2}
!llvm.ident = !{!3}
!0 = !{i32 1, !"wchar\_size", i32 2}
!1 = !{i32 8, !"PIC Level", i32 2}
!2 = !{i32 7, !"uwtable", i32 2}
!3 = !{!"clang version 16.0.4"}
!4 = !{!5, !5, i64 0}
!5 = !{!"any pointer", !6, i64 0}
!6 = !{!"omnipotent char", !7, i64 0}
!7 = !{!"Simple C++ TBAA"}
```

直接使用`clang`加载插件，成功插入解密函数。

![image-20231130155512562](upload/attach/202311/973137_FZJ3M4PRZYE7N5M.png)

![image-20231130160620733](upload/attach/202311/973137_VNZNMGNJY3SKYN4.png)

使用vs加载插件。

![image-20231130161604334](upload/attach/202311/973137_N49738RH7SKZU28.png)
