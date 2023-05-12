
	（include/linux/netdevice.h）
	/**
	 * netif_napi_add() - initialize a NAPI context
	 * @dev:  network device
	 * @napi: NAPI context
	 * @poll: polling function
	 *
	 * netif_napi_add() must be used to initialize a NAPI context prior to calling
	 * *any* of the other NAPI-related functions.
	 */
	// 参数中，dev 为网络设备，napi 为待添加的 napi_struct 结构，poll 为驱动提供的 poll 函数，如 e1000 的为 e1000_clean
	static inline void
	netif_napi_add(struct net_device *dev, struct napi_struct *napi,
	               int (*poll)(struct napi_struct *, int))
	{
	        netif_napi_add_weight(dev, napi, poll, NAPI_POLL_WEIGHT);
	}

调用 netif_napi_add_weight:

	(net/core/dev.c)
	// 参数 weight 保存到 napi_struct->weight，为每次轮询最大处理包的数量，在 netif_napi_add 中传入的是 NAPI_POLL_WEIGHT(64)
	void netif_napi_add_weight(struct net_device *dev, struct napi_struct *napi,
                           int (*poll)(struct napi_struct *, int), int weight)
	{
	        if (WARN_ON(test_and_set_bit(NAPI_STATE_LISTED, &napi->state)))
	                return;
	
	        INIT_LIST_HEAD(&napi->poll_list);
	        INIT_HLIST_NODE(&napi->napi_hash_node);
	        hrtimer_init(&napi->timer, CLOCK_MONOTONIC, HRTIMER_MODE_REL_PINNED);
	        napi->timer.function = napi_watchdog;
	        init_gro_hash(napi);
	        napi->skb = NULL;
	        INIT_LIST_HEAD(&napi->rx_list);
	        napi->rx_count = 0;
			// 设置 poll 函数
	        napi->poll = poll;
	        if (weight > NAPI_POLL_WEIGHT)
	                netdev_err_once(dev, "%s() called with weight %d\n", __func__,
	                                weight);
	        napi->weight = weight;
	        napi->dev = dev;
	#ifdef CONFIG_NETPOLL
	        napi->poll_owner = -1;
	#endif
	        set_bit(NAPI_STATE_SCHED, &napi->state);
	        set_bit(NAPI_STATE_NPSVC, &napi->state);
		    // 挂到 net_device 的 napi_list 链表上
	        list_add_rcu(&napi->dev_list, &dev->napi_list);
	        napi_hash_add(napi);
	        napi_get_frags_check(napi);
	        /* Create kthread for this napi if dev->threaded is set.
	         * Clear dev->threaded if kthread creation failed so that
	         * threaded mode will not be enabled in napi_enable().
	         */
	        if (dev->threaded && napi_kthread_create(napi))
	                dev->threaded = 0;
	}
	EXPORT_SYMBOL(netif_napi_add_weight);
