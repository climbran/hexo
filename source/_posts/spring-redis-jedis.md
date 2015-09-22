title: Spring与redis集成
date: 2015-09-06 13:06:53
tags: [Spring MVC,Redis]
---

首先安装redis：<a href="http://climbran.github.io/2015/09/01/install-redis-in-Mac/">Mac下安装Redis</a>

java客户端选用Jedis，加入pom文件依赖
{% codeblock pom.xml %}
	<dependency>
		<groupId>redis.clients</groupId>
		<artifactId>jedis</artifactId>
		<version>2.7.3</version>
	</dependency>
{% endcodeblock %}
下面是一个简化的Template Demo,用来实现redis各种命令的映射:
{% codeblock com.climbran.redis.JedisTemplate lang:java %}
package com.climbran.redis;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import redis.clients.jedis.JedisPool;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.Pipeline;
import redis.clients.jedis.Tuple;
import redis.clients.jedis.exceptions.JedisConnectionException;
import redis.clients.jedis.exceptions.JedisDataException;
import redis.clients.jedis.exceptions.JedisException;

/**
 * JedisTemplate 提供了一个template方法，负责对Jedis连接的获取与归还。
 * JedisAction<T> 和 JedisActionNoResult两种回调接口，适用于有无返回值两种情况。
 */
public class JedisTemplate {

    private static Logger logger = LoggerFactory.getLogger(JedisTemplate.class);

    private JedisPool jedisPool;

    public JedisTemplate(JedisPool jedisPool) {
        this.jedisPool = jedisPool;
    }

    /**
     * Callback interface for template.
     */
    public interface JedisAction<T> {
        T action(Jedis jedis);
    }

    /**
     * Callback interface for template without result.
     */
    public interface JedisActionNoResult {
        void action(Jedis jedis);
    }

    /**
     * Execute with a call back action with result.
     */
    public <T> T execute(JedisAction<T> jedisAction) throws JedisException {
        Jedis jedis = null;
        boolean broken = false;
        try {
            jedis = jedisPool.getResource();
            return jedisAction.action(jedis);
        } catch (JedisException e) {
            broken = handleJedisException(e);
            throw e;
        } finally {
            closeResource(jedis, broken);
        }
    }

    /**
     * Execute with a call back action without result.
     */
    public void execute(JedisActionNoResult jedisAction) throws JedisException {
        Jedis jedis = null;
        boolean broken = false;
        try {
            jedis = jedisPool.getResource();
            jedisAction.action(jedis);
        } catch (JedisException e) {
            broken = handleJedisException(e);
            throw e;
        } finally {
            closeResource(jedis, broken);
        }
    }

  
    /**
     * Handle jedisException, write log and return whether the connection is broken.
     */
    protected boolean handleJedisException(JedisException jedisException) {
        if (jedisException instanceof JedisConnectionException) {
            logger.error("Redis connection " + jedisPool + " lost.", jedisException);
        } else if (jedisException instanceof JedisDataException) {
            if ((jedisException.getMessage() != null) && (jedisException.getMessage().indexOf("READONLY") != -1)) {
                logger.error("Redis connection " + jedisPool + " are read-only slave.", jedisException);
            } else {
                // dataException, isBroken=false
                return false;
            }
        } else {
            logger.error("Jedis exception happen.", jedisException);
        }
        return true;
    }

    /**
     * Return jedis connection to the pool, call different return methods depends on the conectionBroken status.
     */
    protected void closeResource(Jedis jedis, boolean conectionBroken) {
        try {
            if (conectionBroken) {
                jedisPool.returnBrokenResource(jedis);
            } else {
                jedisPool.returnResource(jedis);
            }
        } catch (Exception e) {
            logger.error("return back jedis failed, will fore close the jedis.", e);
            JedisUtils.destroyJedis(jedis);
        }

    }

    // / Common Actions ///
    public Boolean del(final String... keys) {
        return execute(new JedisAction<Boolean>() {

            @Override
            public Boolean action(Jedis jedis) {
                return jedis.del(keys) == keys.length ? true : false;
            }
        });
    }

    public void flushDB() {
        execute(new JedisActionNoResult() {

            @Override
            public void action(Jedis jedis) {
                jedis.flushDB();
            }
        });
    }

    // / String Actions ///
    public String get(final String key) {
        return execute(new JedisAction<String>() {

            @Override
            public String action(Jedis jedis) {
                return jedis.get(key);
            }
        });
    }

    public void set(final String key, final String value) {
        execute(new JedisActionNoResult() {

            @Override
            public void action(Jedis jedis) {
                jedis.set(key, value);
            }
        });
    }

	//Other Actions
	
	......
}

{% endcodeblock %}

{% codeblock com.climbran.redis.JedisUtils lang:java%}
package com.climbran.redis;

import com.climbran.redis.JedisTemplate.JedisAction;
//import com.climbran.redis.pool.JedisPool;
import redis.clients.jedis.JedisPool;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.exceptions.JedisException;

public class JedisUtils {

	private static final String OK_CODE = "OK";
	private static final String OK_MULTI_CODE = "+OK";

	/**
	 * 判断 返回值是否ok.
	 */
	public static boolean isStatusOk(String status) {
		return (status != null) && (OK_CODE.equals(status) || OK_MULTI_CODE.equals(status));
	}

	/**
	 * 在Pool以外强行销毁Jedis.
	 */
	public static void destroyJedis(Jedis jedis) {
		if ((jedis != null) && jedis.isConnected()) {
			try {
				try {
					jedis.quit();
				} catch (Exception e) {
				}
				jedis.disconnect();
			} catch (Exception e) {
			}
		}
	}
}

{% endcodeblock %}
启动redis服务，执行main函数：
{% codeblock com.climbran.redis.JedisUtils lang:java%}
import com.climbran.redis.JedisTemplate;

import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

public class RedisTest {

    public static void main(String[] args){
        JedisPool pool = new JedisPool(new JedisPoolConfig(),"localhost",6379);
        JedisTemplate template = new JedisTemplate(pool);
        template.set("foo","bar");
        System.out.println(template.get("foo"));
    }
}
{% endcodeblock %}

如果将JedisTemplate注入dao层调用方法则可对entity层与redis进行映射。