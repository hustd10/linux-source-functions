
	(net/core/dev.c)
	/**
	 * alloc_netdev_mqs - allocate network device
	 * @sizeof_priv: size of private data to allocate space for
	 * @name: device name format string
	 * @name_assign_type: origin of device name
	 * @setup: callback to initialize device
	 * @txqs: the number of TX subqueues to allocate
	 * @rxqs: the number of RX subqueues to allocate
	 *
	 * Allocates a struct net_device with private data area for driver use
	 * and performs basic initialization.  Also allocates subqueue structs
	 * for each queue on the device.
	 */
	struct net_device *alloc_netdev_mqs(int sizeof_priv, const char *name,
	                unsigned char name_assign_type,
	                void (*setup)(struct net_device *),
	                unsigned int txqs, unsigned int rxqs)
	
参数：

	sizeof_priv：私有数据域的大小，这个私有数据域紧跟在分配的 net_device 之后，如 tun 设备的私有数据域为 struct tun_struct，对应传入的是 sizeof(struct tun_struct):
				(drivers/net/tun.c)
				dev = alloc_netdev_mqs(sizeof(struct tun_struct), name,
                                       NET_NAME_UNKNOWN, tun_setup, queues,
                                       queues);
	name：网络设备的命名格式，如 "eth%d"
	name_assign_type：网络设备名称的命名方式，定义在：
				（include/uapi/linux/netdevice.h）
				/* interface name assignment types (sysfs name_assign_type attribute) */
				#define NET_NAME_UNKNOWN        0       /* unknown origin (not exposed to userspace) */
				#define NET_NAME_ENUM           1       /* enumerated by kernel */
				#define NET_NAME_PREDICTABLE    2       /* predictably named by the kernel */
				#define NET_NAME_USER           3       /* provided by user-space */
				#define NET_NAME_RENAMED        4       /* renamed by user-space */
	setup: 驱动传入的对 net_device 的初始化函数，例如 loopback 传入的是 loopback_setup()
    txqs：发送队列的数目
	rxqs：接收队列的数目

