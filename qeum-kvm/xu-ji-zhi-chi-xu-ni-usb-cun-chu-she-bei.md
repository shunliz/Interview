USB相关基础

  


USB控制器类型目前有OHCI，UHCI，EHCI，xHCI。OHCI、UHCI都是对应于USB 1.1的标准的，OHCI更多地把要做的事情，用硬件来实现，而UHCI把更多的功能，留给了软件，UHCI更多地应用在PC机中的主板上的USB控制器。EHCI定义了USB 2.0的主机控制器的规范，定义了USB 2.0的主控，xHCI是针对的USB 3.0规范。

  


目前qemu支持1.0、2.0和3.0的控制器模拟，目前使用libvirt工具进行虚机配置启动，在虚机的xml文件中增加相关类型的usb控制器，同时配置disk设备，并使用配置的usb控制器来访问该disk设备即能使虚机访问虚拟usb存储设备。

  


USB存储设备配置

  


要想使用qcow2等镜像文件以u盘方式注入系统，需要配置控制器（controller）以及硬盘（disk）。

  


一

**虚机xml文件中添加USB控制器**

  


**1、ich9类型USB控制器**

  


![](https://mmbiz.qpic.cn/mmbiz_jpg/THOMb1XdkLMWhJJelPpCydJJ9kULDibl0LzaTEBRd1nsicu6mw18N87sXnzHapoEtyYSkkqpaSnsObcq7X8iaXvFg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

可以单独添加ich9-echi1类型的USB2.0控制器，也可以附带ich9-uhcix类型的USB1.0控制器，但不可以只添加USB1.0控制器，ich9类型主控制器为USB2.0控制器，所以必须在xml中写入ich9-ehci1类型控制器，且总线function为0的需加上multifunction='on'。在我的测试机器上使用dd命令写入速度为11.4M/s。  


  


**2、piix3-uhci类型USB1.0控制器**

![](https://mmbiz.qpic.cn/mmbiz_jpg/THOMb1XdkLMWhJJelPpCydJJ9kULDibl0zEnYlmav1Mt0PsWncMScF7YnPlIISSBzaVIfw3Oq8Hmfh2BMMuW8hA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


该类型USB控制器pci必须为0:0:1.2。在我的测试机器上使用dd命令写入速度达到1.2M/s。

  


**3、piix4-uhci类型USB1.0控制器**

**  
**

![](https://mmbiz.qpic.cn/mmbiz_jpg/THOMb1XdkLMWhJJelPpCydJJ9kULDibl0BNvxMFTY3PkjhicYQIibibT1a8Ih1SKM16TCtXHVpL9acI1o4K5LL5PXw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


该USB控制器最好使用新的slot号，，functon为0。测试机器上使用dd命令写入速度达到1.2M/s。

  


**4、vt82c686b-uhci类型USB1.0控制器**

![](https://mmbiz.qpic.cn/mmbiz_jpg/THOMb1XdkLMWhJJelPpCydJJ9kULDibl0NvZzd89dXhq4icFRhoH5aVJnFDYOELXLtuMDLKdu5L5SGyssC5YibZjw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


该USB控制器最好使用新的slot号，functon为0。测试机器上使用dd命令写入速度1.2M/s。

  


**5、ehci类型USB2.0控制器**

**  
**

![](https://mmbiz.qpic.cn/mmbiz_jpg/THOMb1XdkLMWhJJelPpCydJJ9kULDibl0Q5nrhDArrzbic2YngXmVN3RmpgnI3NhpLrhaIJuJj0Qo0TYtkt1ficqQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


该USB控制器最好使用新的slot号，functon为0。测试机器上使用dd命令写入速度达到4M/s。

  


**6、nec-xhci类型USB3.0控制器**

**  
**

![](https://mmbiz.qpic.cn/mmbiz_jpg/THOMb1XdkLMWhJJelPpCydJJ9kULDibl0X9UsgyugbZia47M1IayMbrJwjjZ7QvsngR0Ch6BkIzSUjObMcTrllSA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


由于当前使用vplat系统镜像无该控制器驱动，故不能通过该设置访问USB设备。

  


二

**USB存储盘配置**

  


![](https://mmbiz.qpic.cn/mmbiz_jpg/THOMb1XdkLMWhJJelPpCydJJ9kULDibl0n7UB8ia1ibFiaEczlNV03LjyVvXv9j0TTydZDcIurEiaXecGyMGefon5Pw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

其中source对应的为img文件，将img文件作为U盘设备。

  


目前，libvirt、qemu只支持这些usb控制器，根据对几个的配置进行读写来看正常使用ich9类型读写速度较快，如果vm支持nec-xhci这样的usb3.0控制器也可以使用该配置。整个配置下来就可以在vm中以u盘方式访问img文件，同样可以使用img文件以u盘方式注入来进行u盘启动进行系统安装之类的操作。

