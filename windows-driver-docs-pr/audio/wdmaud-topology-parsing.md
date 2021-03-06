---
title: WDMAud Topology Parsing
description: WDMAud Topology Parsing
ms.assetid: 8aa3e2e8-c9a2-4c3e-94b1-44a0dc218bf3
keywords:
- WDMAud topology parsing WDK audio
- topology parsing WDK audio
- source mixer lines WDK audio
- destination mixer lines WDK audio
- parsing destination mixer lines
- virtual sum WDK audio
- translating nodes WDK audio
ms.author: windowsdriverdev
ms.date: 04/20/2017
ms.topic: article
ms.prod: windows-hardware
ms.technology: windows-devices
---

# WDMAud Topology Parsing


## <span id="wdmaud_topology_parsing"></span><span id="WDMAUD_TOPOLOGY_PARSING"></span>


The [WDMAud system driver](user-mode-wdm-audio-components.md#wdmaud_system_driver) parses destination mixer lines first before parsing the source mixer lines. The order in which WDMAud parses the destination lines is the reverse of that in which SysAudio discovers the lines. For example, the higher numbered pins are parsed first. Parsing starts at the immediate parent of the pin and moves in the upstream direction. Each node is translated according to these rules until the parser detects one of the following terminating conditions:

-   The current node being parsed is a SUM node.

-   The current node is a MUX node.

-   The current node has multiple parents.

SUM and MUX nodes are the *classic terminators* of the destination line. A SUM node does not generate any controls. A MUX node generates a MUX control in the destination line that contains references to each of the source lines controlled by the MUX.

If multiple parents are discovered, parsing is immediately terminated. The mixer-line driver interprets this condition as a "virtual sum" that is formed by tying multiple inputs together.

The name of the destination line comes from the name returned from the [**KSPROPERTY\_PIN\_NAME**](https://msdn.microsoft.com/library/windows/hardware/ff565203) property on that pin.

After all destination line controls have been translated, WDMAud begins translating the source lines. Again, the order in which WDMAud parses these lines is the reverse of the order in which SysAudio queries them. Also, the direction in which source lines are parsed is opposite to that in which destination lines are parsed. WDMAud parses each line starting from the pin and proceeding in the downstream direction until it detects one of the following terminating conditions:

-   The parser finds a destination line.

-   The current node being translated belongs to a destination line.

-   The current node is a SUM node.

-   The current node is a MUX node.

When a MUX is encountered during parsing of a source line that belongs to a destination line, it is translated into a control. However, it is used only as a placeholder to update the line numbers in the MUX stored in the destination line later. The final line numbers are not yet available at this point, so a placeholder is required.

Both a MUX and a SUM node terminate a source line; therefore, any nodes between a SUM or MUX and another SUM or MUX are not translated.

## <span id="Notes"></span><span id="notes"></span><span id="NOTES"></span>Notes


1.  The line names in the MUX are derived from the pin name for the line, except when the line feeding into a MUX is from a SUM or MUX node. In that case, the name of the line is the name of the MUX or SUM node. When the mixer driver discovers this, it builds a virtual mixer line with the name of the SUM or MUX node and then translates all the controls between the SUM or MUX and the MUX.

2.  A *split* in the topology is a case where a node has more than a single child. This is useful when a single pin routes to two separate destinations but shares some common controls, such as volume or a mute. Any time a split is encountered, the WDMAud driver creates a new line and duplicates all the controls parsed up to the split. This happens unconditionally whenever a split is encountered, even after encountering a SUM node that terminates a source line.

 

 


--------------------
[Send comments about this topic to Microsoft](mailto:wsddocfb@microsoft.com?subject=Documentation%20feedback%20[audio\audio]:%20WDMAud%20Topology%20Parsing%20%20RELEASE:%20%287/18/2016%29&body=%0A%0APRIVACY%20STATEMENT%0A%0AWe%20use%20your%20feedback%20to%20improve%20the%20documentation.%20We%20don't%20use%20your%20email%20address%20for%20any%20other%20purpose,%20and%20we'll%20remove%20your%20email%20address%20from%20our%20system%20after%20the%20issue%20that%20you're%20reporting%20is%20fixed.%20While%20we're%20working%20to%20fix%20this%20issue,%20we%20might%20send%20you%20an%20email%20message%20to%20ask%20for%20more%20info.%20Later,%20we%20might%20also%20send%20you%20an%20email%20message%20to%20let%20you%20know%20that%20we've%20addressed%20your%20feedback.%0A%0AFor%20more%20info%20about%20Microsoft's%20privacy%20policy,%20see%20http://privacy.microsoft.com/default.aspx. "Send comments about this topic to Microsoft")


