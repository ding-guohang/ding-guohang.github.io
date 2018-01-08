---
title: 滑动窗口的redis+lua实现
date: 2017-12-24 03:07:29
tags:
- 限流
- redis
categories:
- 笔记
---

lua脚本 + redis，实现滑动窗口用于限流
<!--more-->


>每个key，在redis中是一个table
这个table中记录了多个block的count值， key就是 cnt0, cnt1, ... cnt99 假设100个block



# 算法
计算当前时间所在的block， 计算上一个block
```java
if (时间差已经超过一个duration) {
	全部清空, 重新计数
} else {
	if (当前block > 上一个block) {
		说明两个block中间的block，全部是之前的值，属于无效值，清除掉之后剩下的总数，就是当  前总流量
	}    
        if (当前block < 上一个block) {
                 说明绕了一圈了，上一个block到最后和0到当前block的值，是无效值
        }
}
     
```
其实这个算法，可以通过两份空间，来减少时间上的阻塞
但是因为redis执行lua脚本，本身就是原子的，所以双份空间毫无价值

# 脚本


```lua
-- KEYS[1] map key
-- ARGV[1] current time
-- ARGV[2] duration  窗口总长度
-- ARGV[3] limitation  限流数量
-- ARGV[4] precision  限流精度  duration / 窗口数量10 / 100 * 100  
-- ARGV[5] permits  操作增量 1

local function clear(i1, i2, key, count_key, dele)
    local sum = 0
    for id = i1, i2 do
        local bkey = count_key .. ":" .. id;    -- .. 是字符串拼接的意思
        local bcount = redis.call('HGET', key, bkey)    -- 取出每个block的count值，并累加
        if bcount then
            sum = sum + tonumber(bcount)
            table.insert(dele, bkey)    --  往dele表的末尾插入这些key干啥
        end
    end
    return sum
end

local count_key = "cnt" -- count
local ts_key = "ts" -- timestamp

local key = KEYS[1]
local now = tonumber(ARGV[1])  -- 1513498680  2017/12/17 16:18:00
local duration = tonumber(ARGV[2])
local limit = tonumber(ARGV[3])
local precision = tonumber(ARGV[4])
local permits = tonumber(ARGV[5])

local blocks = math.ceil(duration / precision)  -- 窗口数量 * 100 * 100 这是什么鬼, 会不是默认每个窗口还分为了1w份
local block_id = math.floor(now / precision) % blocks
local last_ts = redis.call('HGET', key, ts_key)  -- hget table field, 所以每个限流的key，是redis中的一个hash表
last_ts = last_ts and tonumber(last_ts) or 0 -- 上一次的时间戳

-- 这个大的if 是计算是否超过limit的
if last_ts ~= 0 then
    local decr = 0;
    local dele = {} -- 创建一个空表
    local last_id = math.floor(last_ts / i) % blocks  -- 不知道这个i是啥，所以我也不知道这个last_id有啥用，但是我猜测应该是上一次落到的block
    local elapsed = now - last_ts;

    if elapsed >= duration then -- 已经是一个新的限流范围了
        -- clear all
        clear(0, blocks - 1, key, count_key, dele)
        if permits > 0 then
            redis.call('HSET', key, ts_key, now)
            redis.call('HINCRBY', key, count_key, permits)  --  总量
            redis.call('HINCRBY', key, count_key .. ":" .. block_id, permits) -- 某个block的量
            redis.call('PEXPIRE', key, duration)
        end
        return false
    elseif block_id > last_id then
        decr = decr + clear(last_id + 1, block_id, key, count_key, dele)  -- 已经过期的值
    elseif block_id < last_id then
        decr = decr + clear(0, block_id, key, count_key, dele)
        decr = decr + clear(last_id + 1, blocks - 1, key, count_key, dele)
    end

    local cur
    if #dele > 0 then -- dele 不为空
        redis.call('HDEL', key, unpack(dele)) -- unpack 返回dele中所有值, 这里是删除这个表中所有失效的block
        cur = redis.call('HINCRBY', key, count_key, -decr)  -- 总量减去过期的数据
    else
        cur = redis.call('HGET', key, count_key)
    end

    if tonumber(cur or '0') + permits > limit then
        return true
    end
end

-- 这个if是增量的
if permits > 0 then
    redis.call('HSET', key, ts_key, now)
    redis.call('HINCRBY', key, count_key, permits)  -- H incr by
    redis.call('HINCRBY', key, count_key .. ":" .. block_id, permits)
    redis.call('PEXPIRE', key, duration)
end
return false
```
