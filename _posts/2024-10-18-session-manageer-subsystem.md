---
title: "Session Manager Subsystem"
lang: ar
date: 2024-10-18 16:56:24 +0800
categories: [Cyber Defense]
tags: [windows]
---
<div dir="rtl">

<h1> نظام إدارة الجلسات (Session Manager Subsystem)</h1>
</div>
<div dir="rtl">
نظام إدارة الجلسات، أو ما يُعرف بملف smss.exe، هو مكون من عائلة أنظمة التشغيل Microsoft Windows NT، بدءًا من Windows NT 3.1. يتم تنفيذه أثناء عملية بدء تشغيل هذه الأنظمة.
</div>
<div dir="rtl">

<h2> تهيئة الجلسات Session initialization</h2>
</div>

<p><div dir="rtl">
يعتبر نظام إدارة الجلسات أول عملية تعمل في وضع المستخدم يتم تشغيلها بواسطة النواة (kernel). بمجرد بدء تشغيله، يقوم بإنشاء ملفات تبديل إضافية (paging files) باستخدام بيانات التهيئة من المسار HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management، ومتغيرات البيئة (environment variables) الموجودة في إدخال التسجيل HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Environment، وخرائط أجهزة DOS (مثل CON: وNUL: وAUX: وCOM1: وCOM2: وCOM3: وCOM4: وPRN: وLPT1: وLPT2: وLPT3:، بالإضافة إلى حروف الأقراص) المدرجة في مفتاح التسجيل
HKLM\System\CurrentControlSet\Control\Session Manager\DOS Devices
يمكن استخدام هذا لإنشاء محركات أقراص دائمة باستخدام subst.
</div>

<p><div dir="rtl">
يتولى مدير الجلسات مسؤولية تشغيل وضع النواة ووضع المستخدم لنظام Win32 الفرعي. يشمل هذا النظام الفرعي ملفات win32k.sys (وضع النواة)، وwinsrv.dll (وضع المستخدم)، وcsrss.exe (وضع المستخدم). كما يتم تشغيل أي أنظمة فرعية أخرى مدرجة في قيمة Required ضمن مفتاح التسجيل 
</div>
<div dir="rtl">
HKLM\System\CurrentControlSet\Control\Session Manager\SubSystems.
</div>

<p><div dir="rtl">
يتولى مدير الجلسات أيضًا تنفيذ أي عمليات يتم طلبها عند بدء الجلسة. يتم تنفيذ الأوامر المدرجة في HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\BootExecute، مثل autochk وconvert، قبل تحميل الخدمات في خطوات بدء التشغيل اللاحقة. كما يتم تنفيذ أي عمليات إعادة تسمية مؤجلة موجودة في HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\PendingFileRenameOperations، مما يسمح باستبدال الملفات التي كانت قيد الاستخدام (مثل برامج التشغيل) كجزء من عملية إعادة التشغيل.
</div>
<p><div dir="rtl">
بدءًا من Windows Vista، يقوم نظام إدارة الجلسات بإنشاء نسخة مؤقتة من نفسه لتشغيل Windows Startup Application (wininit.exe) ونظام Client/Server Runtime Subsystem (csrss.exe) الثاني للجلسة صفر، وهي جلسة مخصصة لعمليات النظام. من هنا، يقوم Windows Startup Application بتشغيل Service Control Manager (services.exe)، الذي يقوم بتشغيل جميع خدمات Windows التي تم تعيينها على "التشغيل التلقائي". كما يقوم التطبيق أيضًا بتشغيل Local Security Authority Subsystem Service (lsass.exe). قبل Windows Vista، كانت هذه العمليات تبدأ بواسطة Windows Logon بدلاً من Windows Startup Application.
</div>
<p><div dir="rtl">
بمجرد تهيئة الجلسة، يقوم نظام إدارة الجلسات بتشغيل  Winlogon (تطبيق تسجيل الدخول إلى Windows)، المسؤول عن التعامل مع تسجيلات الدخول التفاعلية إلى نظام Windows، سواء كانت محلية أو عن بُعد.
</div>

<div dir="rtl">
<h2> العملية Operation</h2>
</div>
<p><div dir="rtl">
بعد انتهاء عملية التمهيد، يبقى البرنامج في الذاكرة ويمكن رؤيته قيد التشغيل في Windows Task Manager. بعد ذلك، ينتظر انتهاء عمليات winlogon.exe أو csrss.exe، وعند حدوث ذلك، سيتم إيقاف تشغيل Windows. إذا لم تنتهِ العمليات بالطريقة المتوقعة، قد يتسبب smss.exe في تعليق النظام، أو قد يحدث خطأ bugcheck. كما يتولى تشغيل جلسات مستخدمين جديدة عند الحاجة.
</div>
<p><div dir="rtl">
يرسل Local Session Manager Service (lsm.exe) طلبات إلى SMSS عبر منفذ ALPC SmSsWinStationApiPort لبدء جلسات جديدة.
في كل مرة يسجل فيها مستخدم دخولًا إلى النظام، يقوم مدير الجلسة بإنشاء نسخة جديدة من نفسه لتكوين جلسة جديدة. تقوم هذه العملية ببدء نظام فرعي جديد Win32 وعملية Winlogon للجلسة الجديدة، مما يتيح للمستخدمين المتعددين تسجيل الدخول في نفس الوقت على أنظمة Windows Server.
</div>

