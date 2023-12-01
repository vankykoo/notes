# Nginx高级

## 一、通过扩容提升整体吞吐量

### 1）整体介绍

通过扩容可以提升程序的吞吐量，也就是系统在单位时间内处理的请求或事务数量。以下是一些方法和注意事项，帮助你通过扩容来提升程序的吞吐量：

1. **水平扩展**：水平扩展是通过增加更多的计算资源来提高吞吐量的一种方法。这可以包括添加更多的服务器、虚拟机、容器实例或者线程来处理更多请求。

2. **负载均衡**：使用负载均衡器来分发请求到多个后端服务器。这可以确保请求均匀分布，避免某些服务器过载，提高系统的整体吞吐量。

3. **数据库优化**：数据库通常是应用程序的瓶颈之一。使用数据库集群、缓存、索引、分区和合适的查询优化来提高数据库性能，从而提高程序的吞吐量。

4. **缓存**：使用缓存来存储频繁访问的数据，以减轻后端服务器和数据库的负担。常见的缓存技术包括内存缓存、分布式缓存和CDN。

5. **异步处理**：将一些计算密集型或IO密集型的任务转换为异步操作，以释放主线程的压力，提高系统的并发能力。

6. **并行处理**：利用多核处理器来并行处理请求。多线程或多进程可以用来同时处理多个请求，提高系统的吞吐量。

7. **微服务架构**：将应用程序拆分成小的微服务，每个微服务独立运行，可以独立扩展。这样可以更容易地增加或减少特定服务的容量。

8. **优化算法**：优化程序的算法和数据结构，以减少不必要的计算和资源消耗，从而提高性能。

9. **监控和调优**：实时监控系统性能，识别瓶颈和性能问题，然后针对性地进行调优。使用性能分析工具来帮助定位问题。

10. **自动化扩展**：使用自动化工具来监测流量并根据需求自动扩展资源，以应对高峰和低谷时期的流量变化。

11. **CDN**：使用内容分发网络（CDN）来缓存静态资源，减轻服务器负担，加速内容传输，提高用户体验。

12. **高可用性**：确保系统具备高可用性，以减少故障期间的性能下降。使用故障转移和备份机制。

13. **限流和熔断**：实施请求限流和熔断机制，以避免系统被过多请求压垮，从而保持系统的稳定性和性能。

总之，通过综合使用这些方法和工具，可以提高程序的吞吐量，确保系统在不断增长的负载下仍能提供高性能和可靠的服务。扩容通常是提高吞吐量的有效手段之一，但也需要根据具体的应用场景和需求来选择合适的扩容策略。



### 2）水平扩容和垂直扩容

垂直扩容和水平扩容是两种不同的系统扩容策略，用于提高应用程序的性能和吞吐量。它们有不同的重点和方法，下面是它们之间的主要区别：

1. **定义**:
   - **垂直扩容（Vertical Scaling）**：垂直扩容是通过增加单个服务器或虚拟机的计算资源来提高系统性能。这可以包括增加CPU、内存、存储容量或网络带宽。垂直扩容的目标是提高单个节点的性能。
   - **水平扩容（Horizontal Scaling）**：水平扩容是通过增加更多的服务器、虚拟机、容器实例或节点来提高系统性能。水平扩容的目标是将负载均匀分散在多个节点上，以处理更多请求。

2. **成本和复杂性**:
   - **垂直扩容**通常较为昂贵，因为需要更强大的硬件，而且有限制。一旦单个节点达到其性能极限，就无法进一步提高性能，且可能需要停机维护。
   - **水平扩容**相对较为经济和可扩展，因为可以不断地添加新的节点来满足需求，而无需中断服务。它也更容易实现高可用性。

3. **故障容忍性**:
   - **垂直扩容**通常较差于故障容忍性，因为单个节点的故障可能导致整个系统宕机。除非有复杂的冗余机制，否则故障恢复时间较长。
   - **水平扩容**通常更有利于故障容忍性，因为多个节点可以分担负载，故障发生时，请求可以继续在其他节点上处理。

4. **性能瓶颈**:
   - **垂直扩容**最终会遇到性能瓶颈，因为单个节点的资源有限，无法无限扩大。对于某些工作负载，这可能会成为问题。
   - **水平扩容**通常具有更高的潜在性能扩展能力，因为可以不断添加新节点，使系统可以更好地应对增长的负载。

5. **适用场景**:
   - **垂直扩容**适用于某些单节点工作负载，例如单一数据库服务器或特定类型的应用程序，可以受益于更大的计算资源。
   - **水平扩容**通常更适用于需要处理大量请求的分布式系统，如Web应用程序、负载均衡的应用程序、大规模数据处理等。

综上所述，选择垂直扩容还是水平扩容取决于应用程序的需求、预算、性能目标和可用性要求。通常情况下，水平扩容更容易实现，更具弹性，并能够更好地应对不断增长的负载。



### 3）水平扩展：集群化

#### 1.ip_hash

在NGINX中，`ip_hash`配置是一种负载均衡策略，用于分发请求到后端服务器，并确保来自相同IP地址的客户端的请求总是被路由到同一个后端服务器，以实现会话保持。这在某些应用程序中非常有用，例如需要保持用户会话一致性的Web应用程序。以下是有关`ip_hash`配置的更多详细信息：

1. **配置`ip_hash`**：
   在NGINX配置文件中，你可以使用`ip_hash`指令来启用IP哈希负载均衡策略。通常，它应该在`http`块中的`upstream`配置中使用。以下是一个示例：

   ```nginx
   upstream backend {
       ip_hash;
       server backend_server1;
       server backend_server2;
       # 添加更多的后端节点
   }
   ```

   在这个示例中，`ip_hash`指令将确保来自相同IP地址的客户端的请求总是被路由到相同的后端服务器。

2. **工作原理**：
   当客户端发起请求时，NGINX将根据客户端的IP地址计算哈希值，并使用哈希值来决定将请求路由到哪个后端服务器。这意味着对于相同IP地址的客户端，它们的请求将始终被路由到相同的后端服务器，从而保持会话的一致性。

3. **注意事项**：
   - `ip_hash`只能保证来自相同IP地址的请求会被路由到同一个后端服务器，它不能处理多个客户端共享相同IP地址的情况（例如，多个用户在同一家公司或学校使用相同的公共IP地址）。
   - 如果你的应用程序依赖于会话状态，请确保会话数据是存储在后端服务器上的，以确保会话的一致性。
   - 注意，`ip_hash`策略可能会导致一些负载不均衡，特别是在某些IP地址下的请求非常频繁，而在其他IP地址下的请求较少的情况下。

4. **维护**：
   请确保你的NGINX配置文件安全，并遵循最佳安全实践，以防止滥用或攻击。

`ip_hash`是一种简单但有效的方式来实现会话保持，特别是在需要保持会话一致性的应用程序中。但要注意它的局限性，并根据你的应用程序需求和环境来选择适当的负载均衡策略。



#### 2.$request_uri配置

在 NGINX 中使用 `hash` 配置 `$request_uri` 变量可以实现一种负载均衡策略，其中请求 URI 的哈希值用于决定将请求路由到哪个后端服务器。这种配置可用于确保相同的请求 URI 总是被路由到相同的后端服务器，以实现会话保持或特定请求的一致性处理。

以下是如何配置 NGINX 使用 `hash` 和 `$request_uri` 实现请求 URI 哈希负载均衡的示例：

```nginx
http {
    upstream backend {
        hash $request_uri;
        server backend_server1;
        server backend_server2;
        server backend_server3;
        # 添加更多的后端服务器
    }

    server {
        location / {
            proxy_pass http://backend;
        }
    }
}
```

在这个示例中：

1. `http` 块中定义了一个名为 `backend` 的负载均衡组，使用 `hash $request_uri;` 配置来启用请求 URI 的哈希负载均衡策略。

2. 后端服务器列表包括 `backend_server1`、`backend_server2`、`backend_server3` 等多个后端服务器。请求 URI 的哈希值将用于确定将请求路由到哪个后端服务器。

3. 在 `server` 块的 `location` 配置中，请求会被代理到 `backend` 负载均衡组。由于使用了 `hash` 配置，相同的请求 URI 将始终路由到相同的后端服务器，实现了请求 URI 的哈希负载均衡。

这种配置适用于需要确保相同的请求 URI 始终被路由到相同的后端服务器的场景，例如在需要保持会话一致性或特定请求的处理一致性方面。然而，要注意以下几点：

- 由于使用哈希函数进行路由，如果后端服务器列表发生变化，将会导致某些请求被重新路由到不同的后端服务器。因此，要确保后端服务器的稳定性和可维护性。

- 当使用哈希负载均衡时，负载均衡的均匀性可能会受到请求 URI 的分布情况的影响。某些 URI 可能会比其他 URI 更频繁地访问，从而导致一些后端服务器的负载更重。在这种情况下，可以考虑使用其他负载均衡策略来平衡负载。

- 对于高流量和高并发的环境，要确保哈希函数和负载均衡配置能够处理请求量，以防止性能瓶颈。



#### 3.$cookie_jsessionid配置

在 NGINX 中，使用 `hash` 配置 `$cookie_jsessionid` 变量可以实现基于会话标识符（通常是 `JSESSIONID` Cookie）的哈希负载均衡策略。这种配置可用于确保具有相同会话标识符的请求总是被路由到相同的后端服务器，从而实现**会话保持**。

以下是如何配置 NGINX 使用 `hash` 和 `$cookie_jsessionid` 实现基于 `JSESSIONID` 的哈希负载均衡的示例：

```nginx
http {
    upstream backend {
        hash $cookie_jsessionid;
        server backend_server1;
        server backend_server2;
        server backend_server3;
        # 添加更多的后端服务器
    }

    server {
        location / {
            proxy_pass http://backend;
        }
    }
}
```

在这个示例中：

1. `http` 块中定义了一个名为 `backend` 的负载均衡组，使用 `hash $cookie_jsessionid;` 配置来启用 `JSESSIONID` Cookie 哈希负载均衡策略。

2. 后端服务器列表包括 `backend_server1`、`backend_server2`、`backend_server3` 等多个后端服务器。`JSESSIONID` Cookie 的值将用于确定将请求路由到哪个后端服务器。

3. 在 `server` 块的 `location` 配置中，请求会被代理到 `backend` 负载均衡组。由于使用了 `hash` 配置，具有相同 `JSESSIONID` Cookie 值的请求将始终路由到相同的后端服务器，实现了会话的一致性。

这种配置适用于需要确保具有相同 `JSESSIONID` Cookie 值的请求始终被路由到相同的后端服务器的场景，以实现会话保持。要注意以下几点：

- 要确保后端服务器可以共享会话数据，以便实现会话保持。通常，会话数据需要存储在后端服务器或共享的存储中。
- 当使用哈希负载均衡时，后端服务器列表的稳定性和可维护性至关重要。如果后端服务器的列表变化频繁，可能会导致一些请求被重新路由到不同的后端服务器。
- 考虑负载均衡的均匀性，以确保不会有某些请求频繁访问具有相同 `JSESSIONID` Cookie 值的后端服务器，从而导致不均匀的负载。
- 对于高流量和高并发的环境，确保哈希函数和负载均衡配置能够处理请求量，以避免性能瓶颈。





###4）keepalive

####1.介绍

HTTP Keep-Alive（也称为持久连接）是一种HTTP协议的特性，**它允许客户端和服务器之间建立一次连接后保持该连接的开放状态，以便在同一连接上传输多个HTTP请求和响应。**这有助于减少每个请求的连接建立和断开的开销，从而提高性能和减少延迟。

以下是有关请求中的Keep-Alive的一些关键信息：

1. **连接重用**：HTTP Keep-Alive允许客户端在单个TCP连接上发送多个HTTP请求，而不是为每个请求建立一个新的TCP连接。这有助于减少网络延迟和资源消耗。

2. **头部字段**：要启用Keep-Alive，客户端和服务器都需要在HTTP请求和响应头中包含`Connection: keep-alive`头部字段。

3. **持续性**：保持连接持续时间可以是有限的，或者它可以一直保持开放状态，直到一个预定的时间或直到发生某个事件。默认情况下，Keep-Alive连接可能会在一段时间后关闭以释放资源，但可以通过`Keep-Alive`头部中的`timeout`参数进行配置。

4. **多个请求**：客户端可以在同一连接上发出多个HTTP请求，服务器将按顺序响应这些请求。这减少了TCP握手和挥手的开销。

5. **HTTP/1.1默认启用**：HTTP/1.1默认启用了Keep-Alive。在HTTP/1.0中，Keep-Alive是可选的，并需要显式启用。

6. **性能优势**：使用Keep-Alive可以提高网站性能，尤其是在有大量资源需要加载的页面，如图像、CSS和JavaScript。通过减少连接建立的次数，可以显著减少页面加载时间。

7. **并发请求**：Keep-Alive允许多个HTTP请求并行处理，而不是按顺序处理。这有助于提高服务器的并发性能。

尽管Keep-Alive提供了性能优势，但它也需要注意以下几点：

- 保持连接可能会占用服务器资源，因此需要适度使用。
- 某些代理服务器和防火墙可能会限制或禁用Keep-Alive连接，因此要确保网络环境允许它。
- 长时间的保持连接可能会导致连接池中的连接堆积，需要进行适当的资源管理和连接超时配置。

总的来说，Keep-Alive是一个有助于提高Web应用性能的功能，但需要在实际应用中进行适当的配置和管理。



####2.使用场景

使用HTTP Keep-Alive（持久连接）还是不使用它取决于你的具体需求和环境。以下是一些情况，你应该考虑使用或不使用Keep-Alive：

**使用HTTP Keep-Alive的情况：**

1. **提高性能**：使用Keep-Alive可以显著提高性能，减少每个请求的连接建立和断开的开销。这对于网站和应用程序的性能至关重要。

2. **多个资源请求**：在页面包含多个资源（如图像、CSS、JavaScript等）的情况下，使用Keep-Alive可以减少资源加载时间，因为多个资源可以并行请求。

3. **减少延迟**：Keep-Alive减少了TCP连接的握手和挥手，从而减少了网络延迟，特别是在高延迟网络上。

4. **减少服务器负载**：与频繁的连接建立和断开相比，使用Keep-Alive可以减少服务器的负载，因为它允许多个请求在同一连接上处理。

5. **支持服务器推送**：在HTTP/2和HTTP/3中，Keep-Alive是默认启用的，支持服务器推送资源，从而提高页面加载性能。

**不使用HTTP Keep-Alive的情况：**

1. **资源受限**：在资源受限的环境中，例如内存或并发连接数受限的服务器，Keep-Alive可能会占用过多资源。在这种情况下，可能需要限制或禁用Keep-Alive。

2. **代理服务器和防火墙限制**：某些代理服务器、防火墙或负载均衡器可能会限制或禁用Keep-Alive连接。在这种情况下，你需要根据网络环境来决定是否使用Keep-Alive。

3. **短暂连接**：对于某些场景，例如只有一个请求的简短连接，保持连接开放可能没有太多优势。在这种情况下，Keep-Alive可能是不必要的。

4. **遗留系统**：一些遗留系统或老旧的浏览器可能不支持Keep-Alive，或者可能会导致问题。你需要根据你的受众来考虑是否启用它。

总的来说，使用Keep-Alive可以提高性能和用户体验，但需要根据特定的环境和需求来配置。在大多数现代Web应用程序中，启用Keep-Alive是一个良好的实践。然而，要确保你的服务器和网络环境能够支持它，以获得最佳性能。



####3.配置

在 NGINX 中配置 Keep-Alive 是一种**使 HTTP 持久连接保持开放状态的方式，以便在同一连接上传输多个 HTTP 请求和响应**。

* **配置 Keep-Alive**：

  在 NGINX 配置中，你需要配置 `keepalive_timeout` 指令。这个指令定义了保持连接持续开放的时间。以下是示例配置：

  ```nginx
  http {
      # 其他配置...
      keepalive_timeout 65s;
    	# 设置为0表示关闭keepalive
  }
  ```

  在这个示例中，`keepalive_timeout` 设置为 65 秒。这意味着连接将在 65 秒后关闭，除非它在此期间收到其他请求。你可以根据你的需求调整这个值。

  现在，NGINX 已配置为使用 Keep-Alive，使连接在一定时间内保持开放状态以处理多个 HTTP 请求。这有助于提高性能并减少延迟。

  要注意，`keepalive_timeout` 的值应根据你的应用程序和服务器性能进行调整。如果设置得太短，可能导致连接频繁关闭和重新建立，从而增加开销。如果设置得太长，可能会占用过多的资源。因此，根据具体情况进行调整以获得最佳性能。

* **keepalive_timeout**：

  此配置项用于**定义每个 Keep-Alive 连接的持续时间，即连接保持打开状态的时间。**默认值通常为 75 秒。你可以根据你的需求进行调整。

  例如，要将 `keepalive_timeout` 设置为 60 秒：

  ```
  nginxCopy code
  http {
      keepalive_timeout 60s;
  }

  ```

  这意味着每个 Keep-Alive 连接将在 60 秒后自动关闭，除非在此期间有新的请求。

* **keepalive_requests**：

  此配置项用于**定义每个 Keep-Alive 连接上可以处理的最大请求数**。当达到此限制时，连接将自动关闭。默认值通常是不限制请求次数。

  例如，要将 `keepalive_requests` 设置为 100：

  ```
  nginxCopy code
  http {
      keepalive_requests 100;
  }

  ```

  这意味着每个 Keep-Alive 连接最多可以处理 100 个请求，然后连接将自动关闭。这可以帮助防止连接过长时间保持开放，释放资源。

  这些配置项通常是在 `http` 块内设置，以全局应用于所有服务器块。但你也可以在特定的 `server` 或 `location` 块中覆盖它们，以适应不同的需求。

  要注意，Keep-Alive 的配置需要根据你的具体服务器性能和需求来进行调整。如果 `keepalive_timeout` 设置得太短，可能会导致连接频繁关闭和重新建立，增加开销。如果 `keepalive_timeout` 设置得太长，可能会占用过多资源。同样，`keepalive_requests` 的值应根据你的应用程序的请求频率进行设置。总之，这些配置项可以帮助你优化性能，但需要根据具体情况进行调整。

* **send_timeout：** 

  用于控制服务器发送响应给客户端的超时时间。具体来说，它定义了服务器等待将响应数据发送到客户端的最长时间。如果在指定的时间内无法发送完响应，服务器将关闭与客户端的连接。这可以有助于避免无限期地挂起连接，从而释放资源并提高服务器的稳定性。



#### 4.上游服务器使用keepalive

在upstream中配置：

```nginx
keepalive 100;
keepalive_timeout 65;
keepalive_requests 1000;
```

> 1. **keepalive 100;**：这是一个配置 `upstream` 块内 Keep-Alive 连接的指令。它指定了与上游服务器之间的 Keep-Alive 连接的最大数量。在这个示例中，最大连接数被设置为 100，这意味着 NGINX 将与上游服务器维护最多 100 个 Keep-Alive 连接。这有助于提高并发性，允许多个客户端同时与上游服务器保持连接。
> 2. **keepalive_timeout 65;**：这个配置项定义了 Keep-Alive 连接的持续时间。具体地，它规定了每个 Keep-Alive 连接在不接收新请求时等待多长时间后被关闭。在这个示例中，`keepalive_timeout` 设置为 65 秒，表示每个连接将在 65 秒不活动后自动关闭。
> 3. **keepalive_requests 1000;**：此配置项定义了每个 Keep-Alive 连接上可以处理的最大请求数。一旦连接处理了指定数量的请求，它将自动关闭。在这个示例中，`keepalive_requests` 设置为 1000，表示每个连接最多可以处理 1000 个请求。



在server的location中配置：

```nginx
proxy_http_version 1.1;
proxy_set_header Connection "";
```

> 1. **proxy_http_version 1.1;**：这是一个用于配置代理服务器的指令。它指定了代理服务器与上游服务器之间使用的 HTTP 版本。在这个示例中，`proxy_http_version` 被设置为 1.1，表示代理服务器将使用 HTTP/1.1 协议与上游服务器进行通信。HTTP/1.1 支持持久连接（Keep-Alive）等性能优化特性，通常用于提高代理服务器的性能。
> 2. **proxy_set_header Connection "";**：这个指令配置了向上游服务器发送的 HTTP 请求头部。具体地，它将请求头部中的 `Connection` 字段设置为空字符串，即 `Connection: ""`。这意味着 NGINX 不会发送 `Connection` 头部给上游服务器。通常，`Connection` 头部用于控制是否使用持久连接（Keep-Alive）。通过将其设置为空字符串，NGINX 表示它不会强制要求使用或禁用持久连接，而是遵循 HTTP 协议的默认行为。



#### 5.在upstream块内外配置keepalive的区别

在 NGINX 中，`keepalive` 配置可以在 `upstream` 块内和 `http` 块外分别进行设置，它们的区别主要在于作用范围和生效的级别：

1. **`keepalive` 配置在 `upstream` 块内**：

   - **作用范围**：`keepalive` 配置在 `upstream` 块内，仅对特定的上游服务器组起作用。这意味着你可以为每个不同的上游服务器组配置不同的 `keepalive` 参数，以满足特定需求。
   - **生效级别**：该配置仅在与指定的 `upstream` 块关联的上游服务器之间生效。其他 `upstream` 块中的上游服务器不受其影响。

   例如：

   ```nginx
   http {
       # ...
       upstream backend_servers {
           server backend_server1;
           server backend_server2;
           keepalive 64; # 针对 backend_servers 配置的 keepalive
       }
   }
   ```

   在此示例中，`keepalive` 配置应用于 `backend_servers` 上游服务器组，而其他上游服务器组不受影响。

2. **`keepalive` 配置在 `http` 块外**：

   - **作用范围**：`keepalive` 配置在 `http` 块外，对整个 NGINX 服务器的所有上游服务器组生效。
   - **生效级别**：该配置对 NGINX 服务器上的所有上游服务器组都生效，除非在单独的 `upstream` 块中覆盖了特定的 `keepalive` 配置。

   例如：

   ```nginx
   http {
       # ...
       keepalive 64; # 对整个 NGINX 服务器上的所有上游服务器组生效
   }

   upstream backend_servers {
       server backend_server1;
       server backend_server2;
       # 这里没有指定 keepalive，将使用全局配置
   }
   ```

   在此示例中，全局 `keepalive` 配置适用于整个 NGINX 服务器，但 `backend_servers` 上游服务器组没有指定特定的 `keepalive` 配置，因此将使用全局配置。

总之，区别在于配置的范围和生效级别。在需要为不同的上游服务器组设置不同的 Keep-Alive 参数时，你可以在 `upstream` 块内进行特定的配置，而在需要为整个 NGINX 服务器的所有上游服务器组设置相同配置时，可以在 `http` 块外进行全局配置。



### 5）使用ab压测nginx

Apache Benchmark（ab）是一个用于执行基准测试和性能测试的命令行工具，通常用于测量 Web 服务器（如 NGINX）的性能。以下是使用 ab 工具来压测 NGINX 的一般步骤：

1. **安装 ab 工具**：

   在大多数 Linux 发行版中，ab 工具通常随 Apache HTTP Server 一起提供。你可以使用以下命令来安装 ab 工具：

   ```bash
   sudo yum install httpd-tools   # 对于 CentOS/Red Hat
   ```

   安装完成后，你可以使用 `ab` 命令来执行压测。

2. **执行 ab 压测**：

   使用以下格式来执行 ab 压测：

   ```bash
   ab -n [总请求数] -c [并发请求数] [URL]
   ```

   - `-n`：指定要执行的总请求数。
   - `-c`：指定并发请求数（同时发送的请求数量）。
   - `[URL]`：指定要测试的目标 URL，通常是 NGINX 服务器的地址和端口号。

   例如：

   ```bash
   ab -n 1000 -c 100 http://localhost/
   ```

   这将发送 1000 个请求，每次并发 100 个请求到 http://localhost/。

3. **分析测试结果**：

   ab 将在测试完成后提供有关测试结果的详细信息，包括每秒请求数、平均响应时间、失败请求数等。结果中的一些关键指标包括：

   - Requests per second (RPS)：每秒请求数，表示服务器每秒能够处理的请求数。
   - Time per request：平均每个请求的响应时间。
   - Failed requests：失败的请求数量。
   - Connect, Receive, and Length columns：连接时间、接收响应时间、内容传输时间的统计信息。

4. **调整测试参数**：

   根据你的测试需求，可以调整总请求数、并发请求数以及测试的 URL。你可以使用不同的参数来模拟不同的测试场景。

注意事项：
- 在进行压测之前，确保 NGINX 服务器已经启动，并且访问的 URL 正确。
- 压测可能会对服务器性能产生影响，因此请小心使用，并在生产环境中谨慎执行。
- 考虑使用负载均衡工具和缓存机制以提高性能。

使用 ab 工具可以帮助你评估 NGINX 服务器的性能，检查其在不同负载条件下的响应能力。在分析测试结果时，可以确定服务器的性能瓶颈，并采取适当的措施进行优化。



### 6）Nginx反向代理核心流程

#### 1.核心流程

Nginx是一款高性能的Web服务器和反向代理服务器，其底层核心流程涉及到多个步骤，用于接收客户端请求，将请求代理到后端服务器，然后将响应返回给客户端。以下是Nginx反向代理的主要底层核心流程：

1. 监听端口：Nginx首先在服务器上监听一个或多个端口，通常是HTTP端口（例如80）或HTTPS端口（例如443）。这些端口用于接收客户端的请求。

2. 接收客户端请求：当客户端发起请求（例如通过浏览器），Nginx接收到该请求。请求通常包括HTTP头和请求正文。

3. 负载均衡（可选）：如果Nginx配置了负载均衡，它可以将请求分发到多个后端服务器，以实现负载均衡和高可用性。Nginx可以使用不同的负载均衡算法，如轮询、IP哈希、最少连接等。

4. 处理请求头：Nginx会检查请求头中的信息，如主机头（Host Header），以确定要将请求代理到哪个后端服务器。这允许Nginx将不同的域名映射到不同的后端服务器。

5. 建立与后端服务器的连接：Nginx建立与选定后端服务器的连接，这可以是HTTP或HTTPS连接，取决于配置。

6. 将请求转发给后端服务器：Nginx将客户端的请求传递给后端服务器，包括请求头和正文。后端服务器会处理请求并生成响应。

7. 接收后端服务器的响应：Nginx接收后端服务器的响应，包括响应头和响应正文。

8. 处理响应头：Nginx可以对后端服务器的响应头进行处理，例如修改响应头中的某些信息，添加HTTP头或修改状态码。

9. 将响应返回给客户端：Nginx将后端服务器的响应传递给客户端，包括响应头和响应正文。

10. 结束连接：一旦响应已经返回给客户端，Nginx会关闭与客户端和后端服务器的连接，以释放资源。

这些是Nginx反向代理的基本底层核心流程。Nginx提供了丰富的配置选项，可以根据需求进行定制，包括缓存、SSL/TLS支持、安全性配置等。这使得Nginx成为一个非常强大和灵活的反向代理服务器，适用于各种不同的应用场景。



#### 2.响应的配置

Nginx允许在接收后端服务器的响应时对响应头进行各种配置和处理。以下是一些可配置的参数和指令，用于控制Nginx在接收后端服务器的响应时的行为：

1. `proxy_set_header`: 这个指令用于设置一个或多个自定义HTTP头字段，将它们添加到Nginx向后端服务器发送的请求中。这可以用于传递一些额外的信息或标识给后端服务器。

```nginx
proxy_set_header X-Custom-Header "Some Value";
```

2. `proxy_hide_header` 和 `proxy_ignore_headers`: 这两个指令用于隐藏或忽略后端服务器响应中的特定HTTP头字段。这对于保护服务器信息和提高安全性很有用。

```nginx
proxy_hide_header Server;
proxy_ignore_headers X-Custom-Header;
```

3. `proxy_pass_header`: 这个指令允许指定一些HTTP头字段，这些字段将被包含在Nginx从后端服务器接收的响应中。通常，这用于确保一些特定头字段在响应中可见。

```nginx
proxy_pass_header X-Backend-Header;
```

4. `proxy_pass_header Set-Cookie`: 这个指令允许将后端服务器响应中的`Set-Cookie`头字段传递到客户端，以便客户端能够接收和处理Cookie信息。

```nginx
proxy_pass_header Set-Cookie;
```

5. `proxy_buffering`: 这个指令用于启用或禁用响应的缓冲。当缓冲启用时，Nginx将在接收完整响应后再将其发送给客户端，而不是实时传递。这可以提高性能。

```nginx
proxy_buffering on;
```

6. `proxy_max_temp_file_size`: 这个指令用于限制Nginx在缓冲响应时写入临时文件的大小。可以防止大文件占用过多磁盘空间。

```nginx
proxy_max_temp_file_size 10m;
```

这些是一些用于配置Nginx在接收后端服务器的响应时处理响应头的常见指令和参数。根据具体需求，你可以在Nginx配置中使用这些指令来自定义和优化反向代理服务器的行为。



#### 3.对客户端请求的缓冲和限制

Nginx提供了一些可配置参数，用于控制客户端请求的缓冲和限制，以确保服务器在面对不良请求或DDoS攻击时能够更好地保护自身资源。以下是一些常见的可配置参数：

**1. `client_body_buffer_size`：**
该指令用于配置Nginx接收客户端请求正文（request body）时的缓冲区大小。默认情况下，Nginx会将请求正文的数据保存在内存中，但如果请求体太大，可以使用该指令控制缓冲区大小。

```nginx
client_body_buffer_size 1K;
```

**2. `client_max_body_size`：**
该指令用于设置允许客户端请求正文的最大大小。如果请求正文超过指定大小，Nginx将拒绝接收请求。这可用于防止服务器受到大型文件上传等请求的攻击。

```nginx
client_max_body_size 10M;
```

**3. `client_body_timeout`：**
这个指令用于设置Nginx等待客户端发送请求正文的超时时间。如果客户端在指定时间内未发送请求正文，连接将被关闭。

```nginx
client_body_timeout 10s;
```

**4. `client_header_buffer_size`：**
该指令用于配置Nginx接收客户端请求头时的缓冲区大小。与`client_body_buffer_size`类似，它用于控制请求头的缓冲区大小。

```nginx
client_header_buffer_size 1K;
```

**5. `client_header_timeout`：**
这个指令用于设置Nginx等待客户端发送请求头的超时时间。如果客户端在指定时间内未发送请求头，连接将被关闭。

```nginx
client_header_timeout 10s;
```

**6. `limit_req` 和 `limit_conn` 模块：**
Nginx还提供了`limit_req`和`limit_conn`模块，用于限制请求的速率和连接数。`limit_req`用于限制每个客户端IP地址的请求速率，而`limit_conn`用于限制每个客户端IP地址的并发连接数。

```nginx
http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
    limit_req zone=one burst=5;
    
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    limit_conn addr 5;
}
```

这些参数和指令可以帮助你控制Nginx如何处理客户端请求，以保护服务器资源免受恶意请求或DDoS攻击的影响。通过调整这些配置，你可以根据你的需求来限制请求的大小、速率和频率。



### 7）获取真实ip

**在使用Nginx作为反向代理服务器时，如果您希望上游服务器获取客户端的真实IP地址，**您需要进行一些配置以传递这些信息。通常，Nginx会将客户端的IP地址传递给上游服务器，通过一些HTTP头或环境变量的方式。

下面是一种常见的方法，通过配置Nginx来传递客户端的真实IP地址给上游服务器：

1. 在Nginx配置中，您需要添加以下配置段到您的`nginx.conf`文件或相关配置文件中：

```nginx
http {
    ...
    
    # 在http块内添加以下配置，以设置真实IP地址
    real_ip_header X-Forwarded-For;
    set_real_ip_from 0.0.0.0/0; # 指定允许的代理IP地址范围，或者使用适当的子网掩码

    ...
}
```

这个配置告诉Nginx将`X-Forwarded-For`请求头中的IP地址视为客户端的真实IP地址。

2. 确保在上游服务器的配置中，它能够识别`X-Forwarded-For`请求头并使用它来获取客户端真实IP地址。具体的配置方式取决于您使用的上游服务器，但通常需要查看上游服务器的文档以了解如何使用`X-Forwarded-For`头。

3. 重新加载Nginx配置以使更改生效：

```bash
sudo systemctl reload nginx
```

现在，当客户端请求通过Nginx反向代理到上游服务器时，上游服务器将能够获取客户端的真实IP地址，这将包含在`X-Forwarded-For`请求头中。

请注意，为了安全性考虑，只有信任的代理应该被允许修改`X-Forwarded-For`头，以防止伪造IP地址。您可以使用`set_real_ip_from`指令来限制允许的代理IP地址范围，以提高安全性。



### 8）gzip压缩

#### 1.介绍

在HTTP传输中使用Gzip是一种数据压缩技术，旨在减小数据传输的大小，提高传输效率。Gzip压缩通常用于HTTP请求和响应，以减少带宽占用和加快页面加载速度。以下是有关如何在HTTP传输中使用Gzip的一些要点：

1. 客户端请求：
   - 当客户端（通常是浏览器）发送HTTP请求时，可以在请求头中包括"Accept-Encoding"字段，指示它支持Gzip压缩。示例请求头：

   ```
   GET /example HTTP/1.1
   Host: www.example.com
   Accept-Encoding: gzip
   ```

   这告诉服务器客户端支持Gzip压缩，服务器可以在响应中使用Gzip进行数据压缩。

2. 服务器响应：
   - 当服务器接收到支持Gzip压缩的请求后，它可以检查响应数据的大小以及服务器是否支持Gzip。如果服务器支持并决定使用Gzip压缩，它会在响应头中添加"Content-Encoding"字段，将其设置为"gzip"，并将响应数据进行压缩。示例响应头：

   ```
   HTTP/1.1 200 OK
   Content-Type: text/html
   Content-Encoding: gzip
   ```

   这告诉客户端响应数据已经使用Gzip进行了压缩。

3. 数据压缩：
   - 在服务器决定使用Gzip压缩后，它将响应数据压缩为Gzip格式。这样，响应体中的文本、图像或其他资源的大小会减小，从而减少传输时间和网络带宽的消耗。

4. 客户端解压：
   - 客户端收到响应后，检查响应头中的"Content-Encoding"字段。如果该字段值为"gzip"，客户端会使用Gzip解压缩算法来还原原始数据，以便进行正常的显示或处理。

使用Gzip压缩可以大幅减小传输数据的大小，提高Web页面加载速度，减少网络带宽占用，从而提高性能和用户体验。几乎所有现代的Web服务器和浏览器都支持Gzip压缩，因此它是一种广泛应用的性能优化技术。



#### 2.参数配置

在使用Nginx作为Web服务器时，你可以配置多个与Gzip压缩有关的参数，以控制响应数据的压缩行为。以下是一些与Gzip相关的常见配置参数：

1. `gzip on`:
   - 该指令用于启用或禁用Gzip压缩。将其设置为"on"以启用Gzip压缩，或设置为"off"以禁用。

   示例：
   ```
   gzip on;
   ```

2. `gzip_types`:
   - 该指令用于指定哪些MIME类型的响应数据应该被压缩。默认情况下，Nginx已经包括了一些常见的MIME类型，但你可以添加或覆盖它们。

   示例：
   ```
   gzip_types text/plain text/html application/json;
   ```
   > **MIME类型，全称是"Multipurpose Internet Mail Extensions"，是一种标识数据的方式，用于指示在互联网上传输的文件的性质和格式。**它最初是为电子邮件系统设计的，以便能够识别和传输不同类型的文件附件。随着时间的推移，MIME类型也被广泛用于Web中，以帮助浏览器和Web服务器正确解释和处理不同类型的文件。
   >
   > MIME类型通常由两部分组成：
   >
   > 1. 主类型（Major Type）：表示文件的大类或性质，如文本、图像、音频、视频等。主类型通常是一个字符串，例如"text"表示文本文件，"image"表示图像文件，"audio"表示音频文件。
   >
   > 2. 子类型（Subtype）：表示文件的具体格式或亚类。子类型通常紧跟在主类型之后，以斜杠（/）分隔。例如，"html"表示HTML文档，"jpeg"表示JPEG图像，"plain"表示纯文本。
   >
   > 综合起来，一个MIME类型的示例是"text/html"，表示这是一个HTML文档，或者"image/jpeg"，表示这是一个JPEG图像。MIME类型在HTTP通信中非常重要，因为它们告诉Web浏览器如何正确解释响应数据，以便显示网页内容或执行其他适当的操作。
   >
   > Web服务器通常在HTTP响应头中使用"Content-Type"字段来指定响应的MIME类型，以便浏览器能够正确处理内容。例如，如果服务器在响应头中设置了"Content-Type: text/html"，浏览器就知道应该将响应解释为HTML文档并进行相应的渲染。

3. `gzip_min_length`:

   - 该指令指定响应数据的最小大小，低于这个大小的数据不会被压缩。这可用于防止对小文件进行不必要的压缩。

   示例：
   ```
   gzip_min_length 100;
   ```

4. `gzip_comp_level`:
   - 该指令用于设置压缩级别，取值范围为1到9，数字越大表示压缩级别越高，但也会消耗更多的CPU资源。默认值为6。

   示例：
   ```
   gzip_comp_level 6;
   ```

5. `gzip_buffers`:
   - 该指令用于设置用于Gzip压缩的内存缓冲区的大小。

   示例：
   ```
   gzip_buffers 16 8k;
   ```

6. `gzip_proxied`:
   - 该指令定义了哪些类型的响应数据应该被压缩，当Nginx作为反向代理时。它支持不同的选项，如"off"、"expired"、"no-cache"等。

   示例：
   ```
   gzip_proxied any;
   ```

7. `gzip_disable`:
   - 该指令允许禁用Gzip压缩的条件。你可以指定某些用户代理（User-Agent）或操作系统，使其不使用Gzip压缩。

   示例：
   ```
   gzip_disable "MSIE [1-6]\.";
   ```

这些是Nginx中与Gzip压缩相关的一些常见配置参数。通过根据你的需求进行适当的配置，你可以优化网站性能，减少带宽占用，提高用户体验。



#### 3.gunzip

在Nginx中，`ngx_http_gunzip_module`是一个模块，它允许服务器在接收到压缩的请求时解压缩内容，以便能够处理经过压缩的数据。这对于一些客户端或代理服务器使用压缩（例如Gzip）来发送请求的情况非常有用。以下是有关Nginx配置的`ngx_http_gunzip_module`的一些要点：

1. 启用`ngx_http_gunzip_module`：
   - 在Nginx中，默认情况下，`ngx_http_gunzip_module`是启用的，但你可以通过在`http`块中的配置中设置以下指令来明确启用它：

   ```nginx
   http {
       gzip off;  # 关闭全局Gzip压缩
       gzip_proxied any;
   }
   ```

   通过关闭全局Gzip压缩，Nginx将只会解压缩来自代理服务器的请求。

2. `gzip_proxied`指令：
   - `gzip_proxied`指令用于指定在哪些情况下应该启用Gzip解压缩。通常，你会将其设置为"any"，以便在任何情况下都启用解压缩。这样，Nginx会尝试解压缩收到的请求，不论请求是否经过代理服务器。

   示例：
   ```nginx
   gzip_proxied any;
   ```

3. 客户端请求：
   - 当Nginx接收到来自客户端的HTTP请求时，如果请求中包括`Accept-Encoding: gzip`等头部字段，表示客户端支持Gzip压缩，Nginx会尝试解压缩请求内容。

4. 响应：
   - 在处理请求后，Nginx会将解压缩后的内容发送给后端应用程序（如果适用），或者继续处理请求。

通过使用`ngx_http_gunzip_module`，Nginx可以在接收到压缩的请求时解压缩内容，以便能够正常处理这些请求。这对于与支持Gzip压缩的客户端或代理服务器进行通信非常有用，确保服务器能够正确解释和处理请求的内容。



```nginx
  gunzip on;
  gzip_static always;
  gzip on;
  gzip_buffers 16 8k;
  gzip_comp_level 6;
  gzip_http_version 1.1;
  gzip_min_length 256;
  gzip_proxied any;
  gzip_vary on;
  gzip_types text/plain application/x-javascript text/css application/xml;
  gzip_types
    text/xml application/xml application/atom+xml application/rss+xml application/xhtml+xml image/svg+xml
    text/javascript application/javascript application/x-javascript
    text/x-json application/json application/x-web-app-manifest+json
    text/css text/plain text/x-component
    font/opentype application/x-font-ttf application/vnd.ms-fontobject
    image/x-icon;
  gzip_disable "MSIE [1-6]\.(?!.*SV1)";
```







### 9）brotli

####1.介绍

Brotli 是一种先进的数据压缩算法，最初由Google开发。它被设计用于减小数据传输的大小，特别适用于互联网上的内容传输，如网页、静态文件、JSON 数据等。Brotli 压缩算法相对于传统的压缩算法（如Gzip）通常能够获得更高的压缩率，这意味着可以减少文件大小，降低网络带宽的使用，并提高网站性能。

以下是一些关于 Brotli 的主要特点和优点：

1. 高压缩率：Brotli 压缩算法通过更先进的压缩技术和算法，通常能够将文件压缩得更小，以减小传输数据的大小。

2. 高性能：Brotli 压缩和解压速度相对较快，尤其是在客户端解压缩方面，这有助于提高网页加载速度。

3. 跨平台支持：Brotli 压缩算法已广泛支持，并且可以在多种操作系统和Web服务器中使用，包括Nginx和Apache。

4. 智能内容类型检测：Brotli 能够根据内容的类型自动选择适当的压缩参数，以便不同类型的数据能够得到最佳的压缩效果。

5. 支持多种文件类型：Brotli 可以用于压缩各种类型的文件，包括HTML、CSS、JavaScript、JSON、XML 等。

6. 流式压缩：Brotli 支持流式压缩，这意味着它可以逐块压缩数据，而不需要等待整个文件可用。

7. 良好的兼容性：现代Web浏览器和服务器通常都支持 Brotli 压缩，这意味着可以广泛应用在Web开发中。

尽管 Brotli 具有许多优点，但它也需要较多的计算资源来进行压缩和解压缩，因此在服务器上使用时需要权衡性能和压缩率。通常，Brotli 用于压缩静态文件，以减小文件大小，提高页面加载速度，并减小网络带宽的使用。在Web开发中，Brotli 常与HTTP的Accept-Encoding头部一起使用，以根据客户端的支持程度动态选择是否应用 Brotli 压缩。



#### 2.使用/配置

下载地址：

- `https://github.com/google/ngx_brotli`
- `https://codeload.github.com/google/brotli/tar.gz/refs/tags/v1.0.9`



在Nginx中配置Brotli压缩，需要使用`ngx_http_brotli_module`模块。以下是一些常用的Brotli配置参数以及它们的说明：

1. `brotli`:
   - 启用或禁用Brotli压缩。可以设置为"on"或"off"。

   示例：
   ```nginx
   brotli on;
   ```

2. `brotli_comp_level`:
   - 设置Brotli压缩的压缩等级，范围是1到11。数字越高表示压缩更强，但也更耗费CPU资源。默认值为6。

   示例：
   ```nginx
   brotli_comp_level 6;
   ```

3. `brotli_types`:
   - 指定应该被Brotli压缩的MIME类型。默认情况下，Nginx已经包括了一些常见的MIME类型。你可以根据需要自定义。

   示例：
   ```nginx
   brotli_types text/html text/plain text/css application/javascript application/json;
   ```

4. `brotli_static`:
   - 启用或禁用Brotli静态文件压缩。当设置为"on"时，Nginx将在静态文件上应用Brotli压缩。

   示例：
   ```nginx
   brotli_static on;
   ```

5. `brotli_comp_level`:
   - 设置Brotli压缩的压缩等级，范围是1到11。数字越高表示压缩更强，但也更耗费CPU资源。默认值为6。

   示例：
   ```nginx
   brotli_comp_level 6;
   ```

6. `brotli_buffers`:
   - 配置用于Brotli压缩的内存缓冲区大小。可以指定缓冲区的数量和大小。

   示例：
   ```nginx
   brotli_buffers 16 8k;
   ```

这些参数可以在Nginx的配置文件中的`http`块内配置。通过适当配置这些参数，你可以启用和自定义Brotli压缩，以减小文件大小、提高网站性能，并降低网络带宽的使用。请确保你的Nginx版本支持Brotli模块，并已正确编译和安装。



```nginx
    brotli on;
    brotli_static on;
    brotli_comp_level 6;
    brotli_buffers 16 8k;
    brotli_min_length 20;
    brotli_types text/plain text/css text/javascript application/javascript text/xml application/xml application/xml+rss application/json image/jpeg image/gif image/png;
```



###10）concat合并请求

Tengine是一个基于Nginx的Web服务器和反向代理服务器，它扩展了Nginx的功能，提供了额外的模块和功能。其中之一是Tengine的`ngx_http_concat_module`模块，用于合并多个HTTP请求中的静态资源（例如CSS和JavaScript文件），以减少浏览器发起的HTTP请求数量，从而提高网页加载性能。以下是有关`ngx_http_concat_module`的详细说明：

1. 启用`ngx_http_concat_module`：
   首先，你需要确保Tengine已经编译并启用了`ngx_http_concat_module`。你可以检查Tengine的编译配置，以确保该模块已经包括在内。通常情况下，Tengine在编译时会默认包括该模块。

2. 配置`concat`指令：
   在Tengine的配置文件中，你需要配置`concat`指令以启用资源的合并。你需要指定要合并的资源类型和文件路径。

   示例：
   ```nginx
   http {
       concat on;
       concat_types text/css text/javascript;
       concat_max_files 10;
       concat_unique on;
   }
   ```

   - `concat on;` 启用合并功能。
   - `concat_types` 指定要合并的资源类型，例如CSS和JavaScript。
   - `concat_max_files` 设置一次合并请求最大允许的文件数量。
   - `concat_unique` 控制是否合并相同文件的多次请求。

3. 使用合并功能：
   一旦合并功能启用，你可以在HTML文件中使用链接合并资源，如下所示：

   ```html
   <link rel="stylesheet" type="text/css" href="/combine.css?file=file1.css,file2.css">
   <script src="/combine.js?file=script1.js,script2.js"></script>
   ```

   这里，`/combine.css`和`/combine.js`是合并请求的URL，`file`参数指定要合并的具体文件。

4. 合并文件路径：
   在Tengine配置中，你需要设置合并文件的存放路径，通常存储在一个合并文件目录中，这个目录必须存在。例如：

   ```nginx
   location /combine/ {
       alias /path/to/your/concat/files/;
   }
   ```

   这个配置将映射`/combine/`路径到你存放合并文件的目录。

使用`ngx_http_concat_module`，你可以有效地减少浏览器的HTTP请求数量，从而提高网页加载性能，减小带宽占用。它特别适用于合并静态资源文件，如CSS和JavaScript，以减小文件传输的大小并提高用户体验。



### 11）资源静态化

####1.介绍

![](C:\Users\86180\Desktop\picPick\235.png)



Nginx 在资源静态化方面具有强大的功能和灵活性。资源静态化是指将动态生成的内容或资源（如HTML、CSS、JavaScript、图像等）缓存为静态文件，以减小服务器负载、提高性能和加速网页加载速度。以下是一些与 Nginx 的资源静态化相关的主要概念和方法：

1. 静态文件缓存：
   - Nginx 可以将静态文件缓存到磁盘，以减少重复请求时的服务器负载。这通常包括 HTML 文件、CSS、JavaScript、图像等。
   - 通过配置 Nginx，你可以指定哪些文件或目录应该被缓存，以及缓存的过期时间。

2. 反向代理缓存：
   - Nginx 可以用作反向代理服务器，它可以缓存动态内容的响应并将其提供为静态文件，以减小对应用服务器的请求负担。
   - 缓存配置可以设置缓存的过期时间、缓存键的生成方式等。

3. FastCGI 缓存：
   - 对于使用 FastCGI 连接的应用程序，Nginx 可以缓存 FastCGI 响应，将其提供为静态文件，从而提高性能。
   - FastCGI 缓存也可以通过 Nginx 配置文件来自定义，以满足不同应用程序的需求。

4. HTTP 缓存头：
   - Nginx 可以在响应中设置合适的 HTTP 缓存头（例如`Cache-Control`、`Expires`、`Last-Modified`等），以控制客户端和代理服务器对资源的缓存行为。

5. 静态文件路径重写：
   - 通过配置 Nginx，你可以将动态 URL 重写为静态文件的路径，以便将请求转发到已缓存的静态资源。
   - 这通常涉及使用`location`块和`try_files`指令。

6. 预渲染：
   - 针对单页面应用（SPA）或需要前端渲染的网站，可以使用预渲染技术，在服务器端生成静态HTML文件以供搜索引擎爬虫和浏览器访问。

7. 静态资源CDN：
   - 使用内容分发网络（CDN）来提供静态资源，以减小资源的传输时间，并将资源缓存在全球各地的CDN节点上。

8. 缓存控制策略：
   - 通过 Nginx 配置，可以设置缓存的策略，包括缓存过期时间、缓存响应码等，以满足特定的应用需求。

Nginx 提供了灵活的配置选项，可以根据具体的应用场景和性能需求来设置资源静态化。使用合适的缓存策略和 Nginx 配置，可以显著减小服务器负载，提高性能，减少带宽占用，并提供更快的用户体验。在设计和配置时，请确保资源静态化不会导致数据不一致的问题，特别是对于经常更新的内容。



####2.ssi介绍

Nginx的SSI（Server Side Includes）模块是一种服务器端包含技术，允许你在Web页面中嵌入动态内容或其他文件的片段。SSI可以用于在静态HTML页面中引入动态数据，如当前日期、文件包含、条件语句等，以便在服务器端进行处理，然后将最终的HTML响应发送给客户端。以下是关于Nginx的SSI模块的一些关键概念和用法：

1. 启用SSI模块：
   要使用SSI功能，你需要在Nginx配置文件中启用SSI模块。通常，这可以通过`ssi on;`指令来实现。例如：

   ```nginx
   location / {
       ssi on;
   }
   ```

   这将在指定的`location`中启用SSI处理。

2. SSI指令：
   SSI模块支持多种指令，用于在HTML文件中插入动态内容。以下是一些常见的SSI指令示例：

   - `<!--#include virtual="/path/to/file.html" -->`: 用于包含另一个HTML文件的内容。
   - `<!--#echo var="DATE_LOCAL" -->`: 显示本地日期和时间。
   - `<!--#if expr="$QUERY_STRING = /admin/" -->`: 根据条件表达式包含不同的内容。

   你还可以自定义SSI指令，以便在HTML中执行自定义操作。

3. 条件语句：
   SSI模块支持条件语句，可以根据条件的满足情况包含不同的内容。例如：

   ```html
   <!--#if expr="$QUERY_STRING = /admin/" -->
   <p>Welcome, admin!</p>
   <!--#else -->
   <p>Welcome, guest!</p>
   <!--#endif -->
   ```

   在这个示例中，根据请求的查询字符串，SSI会包含不同的欢迎消息。

4. 变量和环境变量：
   SSI模块支持内置变量，如`$DOCUMENT_NAME`、`$HTTP_USER_AGENT`等，以及可以自定义的环境变量，这些变量可以在SSI指令中使用。

5. 缓存：
   SSI模块可以与Nginx的缓存模块一起使用，以提高性能。它可以根据需要在缓存内容中包含SSI指令，然后在从缓存中提供响应时执行SSI。

6. 配置文件：
   你可以在Nginx配置文件中指定SSI的配置选项，如SSI的起始标记和结束标记。默认情况下，SSI的起始标记是`<!--#`，结束标记是`-->`。

   ```nginx
   ssi on;
   ssi_silent_errors on;
   ssi_types text/html;
   ssi_start "<!--#";
   ssi_end "-->";
   ```

SSI模块为Web开发者提供了在静态HTML页面中引入动态内容的灵活性，而无需使用服务器端脚本语言。**这对于包含公共页眉、页脚、导航栏等元素的网站非常有用，因为这些元素可以在所有页面中共享，从而减小维护工作。**需要注意的是，SSI通常用于较简单的动态内容，如果需要更复杂的服务器端逻辑，可能需要使用其他服务器端技术。



####3.ssi常用配置

Nginx SSI（Server Side Includes）模块支持多个可配置参数，这些参数可以用来控制SSI的行为和功能。以下是一些常见的可配置参数：

1. `ssi on | off`：
   - 用于启用或禁用SSI模块。如果设置为`on`，则启用SSI处理，允许在HTML页面中使用SSI指令。如果设置为`off`，则禁用SSI。

   示例：
   ```nginx
   ssi on;
   ```

2. `ssi_types`：
   - 用于指定哪些响应的内容类型应该由SSI模块进行处理。通常，这将设置为`text/html`，以便只对HTML响应进行SSI处理。

   示例：
   ```nginx
   ssi_types text/html;
   ```

3. `ssi_include_error`：
   - 用于控制当SSI包含文件出错时的行为。如果设置为`on`，SSI错误将被包含并显示在页面上。如果设置为`off`，则错误不会显示在页面上。

   示例：
   ```nginx
   ssi_include_error on;
   ```

4. `ssi_silent_errors`：
   - 用于控制SSI错误时是否在Nginx的错误日志中记录错误信息。如果设置为`on`，则错误信息会被记录；如果设置为`off`，则错误信息不会记录。

   示例：
   ```nginx
   ssi_silent_errors off;
   ```

5. `ssi_start` 和 `ssi_end`：
   - 用于自定义SSI指令的起始和结束标记。默认情况下，SSI指令使用`<!--#`作为起始标记，`-->`作为结束标记。你可以自定义这些标记以适应特定的需求。

   示例：
   ```nginx
   ssi_start "<!--#";
   ssi_end "-->";
   ```

这些可配置参数允许你根据具体的需求来控制SSI模块的行为。通过配置这些参数，你可以定制SSI的功能，以满足特定应用的要求。要记住，配置SSI参数时需要小心，以确保安全性和性能。



#### 4.ssi常用指令

Nginx的SSI（Server Side Includes）模块支持多个指令，用于在HTML页面中嵌入动态内容或执行不同的操作。以下是一些常用的SSI指令：

1. `<!--#include virtual="path" -->`：
   - 用于包含其他文件的内容。`virtual`属性指定了要包含的文件的路径。可以是绝对路径或相对路径。

   示例：
   ```html
   <!--#include virtual="/includes/header.html" -->
   ```

2. `<!--#echo var="variable" -->`：
   - 用于显示SSI变量的值。`variable`属性指定了要显示的变量名称。

   示例：
   ```html
   <!--#echo var="DATE_LOCAL" -->
   ```

   内置的SSI变量包括`DATE_GMT`、`DATE_LOCAL`、`DOCUMENT_NAME`、`HTTP_USER_AGENT`等。

3. `<!--#if expr="condition" -->` 和 `<!--#elif expr="condition" -->`：
   - 用于创建条件语句，根据条件的满足情况包含不同的内容。

   示例：
   ```html
   <!--#if expr="$QUERY_STRING = /admin/" -->
   <p>Welcome, admin!</p>
   <!--#elif expr="$QUERY_STRING = /user/" -->
   <p>Welcome, user!</p>
   <!--#else -->
   <p>Welcome, guest!</p>
   <!--#endif -->
   ```

4. `<!--#set var="variable" value="content" -->`：
   - 用于设置自定义SSI变量的值，以便在页面中重复使用。

   示例：
   ```html
   <!--#set var="my_var" value="Hello, SSI!" -->
   <!--#echo var="my_var" -->
   ```

5. `<!--#config`：
   - 用于配置SSI的参数，如`<!--#config errmsg="404 Not Found" -->`可用于设置在包含文件出错时的错误消息。

6. `<!--#exec cmd="command" -->`：
   - 用于执行系统命令，并将命令的输出插入到页面中。这是一个潜在的安全风险，应该谨慎使用，并确保只执行受信任的命令。

   示例：
   ```html
   <!--#exec cmd="date" -->
   ```

这些是SSI模块中的一些常用指令。SSI使得可以在静态HTML页面中引入动态内容，从而提高了页面的可重用性和动态性。要注意SSI指令的安全性，确保只允许受信任的用户或者对指令进行了适当的限制，以避免潜在的安全风险。



#### 5.rsync介绍

rsync（Remote Sync）是一种**用于文件和目录同步**的强大工具，广泛用于在本地或远程计算机之间快速、安全地传输和同步数据。rsync的特点包括：

1. **快速传输**：rsync通过使用增量传输和复制差异数据的方式，可以极大地提高文件传输的速度。它只传输发生变化的部分，而不是整个文件，从而减小了传输的数据量。

2. **强大的文件同步**：rsync支持文件和目录的同步，可以在本地或远程计算机之间同步文件和目录结构。这使得它在备份、数据迁移和文件分发等场景中非常有用。

3. **多种传输协议**：rsync支持多种传输协议，包括本地文件系统、SSH、rsync协议、和其他网络协议。这使得它能够灵活地应用于不同环境。

4. **安全传输**：通过与SSH一起使用，rsync可以实现加密传输，保护数据的安全性。

5. **保持权限和时间戳**：rsync可以保持文件和目录的权限、时间戳、软链接等信息，确保在目标端的文件与源端保持一致。

6. **部分传输和断点续传**：如果传输中断，rsync可以从中断处继续，而不必重新开始传输。

7. **多种操作模式**：rsync可以用于文件复制、备份、镜像、数据同步、远程文件分发等多种操作模式。

8. **筛选和排除**：你可以使用rsync的筛选和排除规则，指定哪些文件和目录应该传输或排除，以满足特定的同步需求。

9. **脚本和自动化**：rsync可以轻松地集成到脚本和自动化工作流程中，以执行定期的文件同步任务。

10. **开源和跨平台**：rsync是开源的，并可以在多种操作系统上运行，包括Linux、Unix、Windows等。

使用rsync通常涉及指定源目录或文件，以及目标目录或文件，然后执行rsync命令。例如：

```bash
rsync -avz /path/to/source/ user@remote_server:/path/to/destination/
```

这个命令将从本地`/path/to/source/`同步到远程服务器`remote_server`的`/path/to/destination/`目录。 `-a`表示使用归档模式进行同步， `-v`表示启用详细输出， `-z`表示使用压缩传输。

rsync在许多情况下是一个非常有用的工具，特别是在需要在不同计算机之间同步文件和目录时。它提供了高效、灵活和安全的文件传输和同步机制。

![](C:\Users\86180\Desktop\picPick\236.png)



Nginx中并没有内置的rsync工具，但rsync是一种强大的文件同步工具，可以与Nginx服务器一起使用，以便快速、可靠地同步文件和目录。以下是如何在Nginx服务器中使用rsync工具的一般步骤：

1. 安装rsync：
   首先，确保你的服务器上安装了rsync。你可以使用系统的包管理工具来安装rsync。例如，在Ubuntu上，你可以运行以下命令：

   ```bash
   sudo apt-get install rsync
   ```

2. 创建rsync配置文件：
   创建一个rsync的配置文件以定义文件同步的规则和目标位置。通常，rsync配置文件的路径是`/etc/rsyncd.conf`。你可以根据需要配置不同的模块，每个模块定义了一组同步规则和目标位置。

   示例rsync配置文件：

   ```bash
   uid = nobody
   gid = nogroup
   use chroot = yes
   max connections = 4
   syslog facility = local5

   [web]
   path = /var/www/html/
   comment = Nginx Web Directory
   read only = no
   ```

   在这个示例中，我们定义了一个名为"web"的rsync模块，它将同步本地`/var/www/html/`目录到远程服务器。

3. 启动rsync服务：
   启动rsync服务以便在Nginx服务器和其他服务器之间同步文件。你可以使用以下命令启动rsync服务：

   ```bash
   sudo rsync --daemon
   ```

4. 使用rsync命令同步文件：
   在Nginx服务器上，你可以使用rsync命令来同步文件和目录。以下是一个示例命令：

   ```bash
   rsync -avz /path/to/local/files/ remote_server::web/
   ```

   - `-a`：表示以归档模式同步，保留文件属性和权限。
   - `-v`：启用详细输出，以便查看同步进度。
   - `-z`：使用压缩传输。
   - `--delete`  ：**删除目标目录比源目录多余文件。**
   - `/path/to/local/files/`：本地文件或目录的路径。
   - `remote_server::web/`：远程rsync模块的目标位置。

这是一个基本的使用rsync同步文件的示例。你可以根据实际需求和安全性设置来配置rsync和Nginx服务器，以确保安全地同步文件。请注意，rsync可以通过SSH进行加密传输，这是一个常见的用例，特别是在跨网络进行文件同步时。



#### 6.同步方案

* 近时同步

  * 目标服务器定时到源服务器检查有没有新增或删除的资源文件，如果有，就**拉**过来/删除。

* 实时同步

  * 源服务器的资源文件有变化就**推**到目标服务器中。

  ①源服务器需要配置

  ```conf
  read only = no
  ```

  ②目标服务器：

  ```shell
  rsync -avz --password-file=/etc/rsyncd.pwd.client /usr/local/nginx/html/ vanky@192.168.200.140::ftp/
  ```

  * 使用脚本进行监听

  在源服务器中创建一下sh脚本，并执行。【192.168.200.140是目标服务器。】

  ```shell
  #!/bin/bash

  /usr/local/inotify/bin/inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format '%T %w%f %e' -e close_write,modify,delete,create,attrib,move //usr/local/nginx/html/ | while read file
  do

          rsync -az --delete --password-file=/etc/rsyncd.pwd.client /usr/local/nginx/html/ vanky@192.168.200.140::ftp/
  done
  ```

  ​

> 要使用inotify和rsync实现Nginx某个文件夹的实时同步，你可以利用inotify监控文件夹的变化，并在文件发生更改时触发rsync命令来进行同步。以下是一般的步骤：
>
> 1. 安装inotify-tools：
>    在Linux系统上，你需要安装inotify-tools，它包含了一些工具，用于监控文件系统的变化。你可以使用包管理器来安装它，如：
>
>    ```bash
>    sudo apt-get install inotify-tools  # 对于Ubuntu/Debian
>    sudo yum install inotify-tools      # 对于CentOS/Red Hat
>    ```
>
> 2. 创建一个脚本：
>    创建一个shell脚本，用于监控文件夹并在文件变化时触发rsync同步。以下是一个示例脚本，你可以根据实际需要进行自定义：
>
>    ```bash
>    #!/bin/bash
>
>    SOURCE_DIR="/path/to/source/directory"
>    DEST_DIR="/path/to/destination/directory"
>    RSYNC_OPTIONS="-avz"  # 根据需要设置rsync的选项
>
>    while inotifywait -e modify,create,delete -r "$SOURCE_DIR"; do
>        rsync $RSYNC_OPTIONS "$SOURCE_DIR" "$DEST_DIR"
>    done
>    ```
>
>    在这个脚本中，`inotifywait`命令用于监控`$SOURCE_DIR`目录中的文件修改、创建和删除事件，然后执行rsync命令来同步变化的文件到`$DEST_DIR`目录。
>
> 3. 为脚本添加执行权限：
>    使用`chmod`命令为脚本添加执行权限：
>
>    ```bash
>    chmod +x your_sync_script.sh
>    ```
>
> 4. 启动脚本：
>    运行脚本，开始实时同步：
>
>    ```bash
>    ./your_sync_script.sh
>    ```
>
> 现在，当Nginx某个文件夹中的文件发生更改时，脚本将自动触发rsync命令来实现实时同步。请确保在脚本中正确设置源目录、目标目录和rsync选项以满足你的需求。
>
> 请注意，这只是一个基本示例，实际需求可能需要更复杂的脚本和处理，特别是在需要处理大量文件和更多同步选项时。确保你的脚本能够处理各种情况，如文件冲突、同步速度等。





### 12）多级缓存

#### 1.介绍

浏览器中也使用了多级缓存以提高网页加载性能和用户体验。这些缓存层次通常包括以下几个：

1. **浏览器缓存**：浏览器会在本地缓存已访问过的网页资源，例如HTML、CSS、JavaScript文件、图像等。这可以降低网页加载时间，因为浏览器不需要再次下载已经存在于本地缓存中的资源。

2. **内存缓存**：浏览器使用内存缓存存储临时数据，如JavaScript运行时数据、页面状态信息等。内存缓存速度非常快，但容量有限，通常随着浏览器标签的关闭而清空。

3. **磁盘缓存**：浏览器还将一些数据存储在磁盘缓存中，例如已下载的资源、浏览历史等。这些数据可以在会话之间保持持久性，以减少网络请求和提高网页加载速度。

4. **HTTP缓存**：HTTP缓存是基于HTTP协议的缓存机制，它允许浏览器和服务器之间协商资源是否需要重新下载。浏览器会发送请求中的缓存控制头（如`Cache-Control`和`ETag`），以指示服务器何时更新资源。如果资源没有发生更改，服务器可以返回状态码304，告知浏览器使用本地缓存。

5. **CDN缓存**：内容分发网络（CDN）提供全球性的缓存服务，它们分布在多个地理位置上，允许浏览器从最接近用户的CDN节点获取资源。CDN缓存可以加速资源的传递，减少延迟和提高性能。

这些多级缓存层次帮助浏览器更快地加载网页，并减少对服务器的请求次数，从而提高了用户体验。同时，浏览器还可以通过清除缓存或强制刷新来处理特定情况下的缓存问题，以确保用户能够访问最新的网页内容。浏览器的多级缓存系统在一定程度上也有助于减少对网络带宽和服务器资源的需求，从而减轻了服务器的负载。



####2.强制缓存和协商缓存

强制缓存和协商缓存是Web浏览器和服务器之间控制缓存的两种主要方法。它们帮助提高网页性能并减少网络资源的重复下载，从而减少了带宽消耗。下面是对这两种缓存策略的介绍：

1. **强制缓存**：

   强制缓存是一种缓存策略，服务器在响应资源请求时，通过HTTP响应头来告诉浏览器，该资源在一段时间内是不会发生变化的。浏览器在第一次请求资源时会缓存该资源，并在之后的请求中直接从缓存中获取，而不必再次请求服务器。这节省了带宽和加速了页面加载。

   强制缓存通常使用以下HTTP响应头字段来设置：
   - `Cache-Control`：可以包含`max-age`指令，指定资源的最长缓存时间（以秒为单位）。
   - `Expires`：指定资源的过期日期和时间，即资源将在该日期之后失效。
   - `ETag`：是资源的标识符，用于标记资源的版本。

2. **协商缓存**：

   协商缓存是一种缓存策略，服务器在资源请求中包含条件，浏览器通过发送请求时的条件来询问服务器资源是否已经发生了变化。如果资源没有发生变化，服务器返回一个状态码304（未修改）而不是资源本身，浏览器接着从本地缓存获取资源。

   协商缓存通常使用以下HTTP头字段来实现：
   - `If-Modified-Since`：浏览器将资源的最后修改时间发送给服务器。
   - `If-None-Match`：浏览器将资源的ETag发送给服务器。

   服务器根据这些条件来判断是否需要提供新的资源或返回304状态码。

强制缓存和协商缓存通常是同时存在的，浏览器首先会检查强制缓存，如果资源在强制缓存期间内，则直接从本地缓存中获取，不会向服务器发送请求。如果资源超过了强制缓存期限，浏览器会发送条件请求，根据协商缓存策略来判断是否需要从服务器获取新的资源。

这两种缓存策略在不同情况下有不同的应用，强制缓存通常用于资源稳定不经常变化的情况，而协商缓存用于资源可能会经常变化的情况。合理配置缓存策略可以有效提高网页性能，并降低服务器和网络的负担。



**强制缓存开启时，只有在第一次进入页面时会使用缓存，刷新之后会重新发送请求。**

**协商缓存开启时，如果页面没有改变，就会使用缓存。**



> 关闭方法：
>
> * 关闭last-modified，关闭etag
>
>   ```nginx
>   location / {
>     etag off;
>     add-header Last-Modified "";
>     
>     
>     #   root   html;
>     index  index.html index.htm;
>   }
>   ```
>
> * 关闭etag，关闭if_modified_since【发送但不理会】
>
>   ```nginx
>   location / {
>     etag off;
>     # add-header Last-Modified "";
>     if_last_modified off;
>     
>     #   root   html;
>     index  index.html index.htm;
>   }
>   ```



> 配置强制缓存：
>
> * expires
> * add_header cache-control "max-age:300";



在Nginx中，您可以使用一些配置参数来管理强制缓存和协商缓存的行为。以下是与这两种缓存策略相关的一些重要Nginx配置指令：

**1. 强制缓存相关配置参数：**

- `expires`：这是一个用于设置资源过期时间的指令，它可以告诉浏览器资源应该被缓存多长时间。您可以配置不同的`expires`指令来设置不同类型的资源的过期时间，例如静态文件、HTML页面等。

```nginx
location ~* \.(css|js|jpg|jpeg|png)$ {
    expires 30d;
}
```

- `add_header`：使用`add_header`指令，您可以添加自定义HTTP响应头，包括`Cache-Control`和`Expires`。这使您能够更细粒度地控制资源的缓存策略。

```nginx
location ~* \.(css|js|jpg|jpeg|png)$ {
    expires 30d;
    add_header Cache-Control "public, max-age=2592000";
}
```

**2. 协商缓存相关配置参数：**

- `etag`：`etag`指令用于生成资源的ETag标识符。ETags通常用于协商缓存，浏览器可以通过发送`If-None-Match`请求头来检查资源是否已更改。

```nginx
location ~* \.(css|js|jpg|jpeg|png)$ {
    etag on;
}
```

- `if_modified_since`：`if_modified_since`指令用于启用资源的“Last-Modified”标头，这有助于实现协商缓存。浏览器会发送`If-Modified-Since`请求头来检查资源是否已更改。

```nginx
location ~* \.(css|js|jpg|jpeg|png)$ {
    if_modified_since exact;
}
```

- `open_file_cache`：`open_file_cache`指令允许您配置Nginx在打开文件时缓存文件描述符，这可以提高性能，特别是在大量请求时。

```nginx
open_file_cache max=1000 inactive=20s;
```

这些是一些在Nginx中配置强制缓存和协商缓存的常见指令。您可以根据特定的应用和需求进行更详细的配置。在配置Nginx缓存时，务必谨慎，以确保您的站点性能和缓存策略按预期工作。





### 13）CDN

CDN，即内容分发网络（Content Delivery Network），是一种**用于提高网络内容传输效率和性能的技术和服务**。CDN的主要目标是**通过将内容分发到位于全球各地的多个服务器节点，以减少用户请求内容时的延迟和提高内容的可用性和可靠性。**

以下是CDN的一些关键特点和工作原理：

1. **缓存和复制**：CDN将网站的静态和动态内容（如文本、图像、视频、JavaScript等）缓存到多个服务器节点上，以减少从源服务器传输内容的需求。这些缓存副本被分布在世界各地的数据中心，以减少距离和网络拥塞对内容传输速度的影响。

2. **负载均衡**：CDN使用负载均衡技术，将用户请求路由到离用户最近的服务器节点，从而降低响应时间，并提供更好的性能和可用性。

3. **自动缓存更新**：CDN通常支持内容自动更新，确保用户获得最新的内容，而不必等待缓存过期。这可以通过将内容缓存在CDN节点上并与源服务器同步来实现。

4. **安全性**：CDN可以提供安全性增强，如DDoS攻击防护、SSL加密、WAF（Web应用程序防火墙）等功能，以保护网站免受各种网络威胁的侵害。

5. **可伸缩性**：CDN具有高度可伸缩性，可以根据流量需求自动扩展或缩减服务器节点，以满足不断变化的流量需求。

6. **数据分析和监控**：CDN服务通常提供流量统计、性能监控和报告功能，帮助网站管理员了解用户行为和性能指标，从而优化其网站和内容分发策略。

CDN的应用范围广泛，从网站加速和内容传输到视频流媒体和应用程序交付等多个领域都有应用。通过降低加载时间、提高内容可用性和提供更好的用户体验，CDN在全球范围内成为了许多互联网企业的重要组成部分。



### 14）Geoip2

#### 1.介绍

GeoIP2 是 MaxMind 公司开发的一种IP地理位置信息库，用于确定特定IP地址的地理位置信息。与传统的 GeoIP 数据库相比，GeoIP2 提供了更准确、更详细的地理位置数据，包括国家、地区、城市、邮政编码、经纬度、时区、ISP（互联网服务提供商）等信息。GeoIP2 数据库的更新频率通常比传统 GeoIP 数据库更高，因此它可以提供更精确的IP地址定位信息。

以下是一些关于 GeoIP2 的主要特点和用途：

1. 更精确的地理位置信息：GeoIP2 提供了更准确的地理位置信息，可以在更细粒度的级别上确定IP地址的位置，包括城市级别的信息，这对于内容定制和地理定位非常有用。

2. 高度详细的数据：GeoIP2 数据库包含了大量的额外信息，如时区、经纬度坐标、邮政编码等，使其在各种应用中的用途更加广泛。

3. 多种数据格式支持：GeoIP2 数据库支持多种数据格式，包括二进制格式和CSV格式，以满足不同应用程序的需求。

4. 定期更新：GeoIP2 数据库通常会定期更新，以反映新的IP地址分配和地理位置信息的变化，确保数据的准确性。

5. 应用领域广泛：GeoIP2 数据库可用于多种用途，包括内容分发、广告定位、访问控制、网络安全、统计分析等。

6. API支持：MaxMind还提供了用于访问 GeoIP2 数据的API，使开发人员能够轻松地在自己的应用程序中使用这些数据。

7. 支持IPv4和IPv6：GeoIP2 数据库不仅支持IPv4地址，还支持IPv6地址，使其能够适用于未来的互联网发展。

总的来说，GeoIP2 是一种强大的工具，可以帮助网站、应用程序和服务更好地了解其用户的地理位置，以实现地理定位、内容定制、访问控制、广告定位和统计分析等用途。它对于全球定位和个性化体验的需求非常有价值。



#### 2.使用

在 Nginx 中使用 GeoIP2 模块可以帮助你根据客户端的IP地址获取地理位置信息，并根据这些信息来进行访问控制、内容定制、统计分析等操作。下面是一些使用 GeoIP2 模块的基本步骤：

1. 安装 GeoIP2 数据库：
   - 首先，你需要获取 GeoIP2 数据库文件，这些文件包含了IP地址与地理位置信息的映射。你可以从 MaxMind（https://dev.maxmind.com/geoip/geoip2/downloadable/） 下载免费的 GeoIP2 数据库，或者根据你的需求订阅他们的付费服务以获得更详细的数据。

2. 安装 GeoIP2 模块：
   - GeoIP2 模块是 Nginx 的第三方模块，你需要编译 Nginx 时包含该模块。通常，你需要在编译 Nginx 之前指定 GeoIP2 模块的路径，以确保 Nginx 支持 GeoIP2 功能。示例：

     下载地址：https://github.com/leev/ngx_http_geoip2_module
   ```
   ./configure --with-http_geoip_module --with-stream_geoip_module --add-dynamic-module=/path/to/ngx_http_geoip2_module
   ```

3. 配置 Nginx 使用 GeoIP2 模块：
   - 在 Nginx 的配置文件中，你可以使用 GeoIP2 模块的指令来配置如何使用地理位置信息。以下是一些常见的配置示例：

   ```nginx
   http {
       geoip2 /path/to/GeoIP2-City.mmdb {
           $geoip2_data_continent continent code;
           $geoip2_data_country country iso_code;
           $geoip2_data_region region iso_code;
           $geoip2_data_city city names en;
           $geoip2_data_postal_code postal code;
           $geoip2_data_latitude latitude;
           $geoip2_data_longitude longitude;
       }

       server {
           location / {
               if ($geoip2_data_country = "US") {
                   # 如果用户位于美国，执行某些操作
               }
           }
       }
   }
   ```

4. 使用 GeoIP2 变量：
   - 在配置中，你可以使用 GeoIP2 变量来访问获取的地理位置信息。你可以在 Nginx 的 `if` 指令中使用这些变量来进行条件判断，以实现访问控制或其他操作。

5. 重启 Nginx：
   - 最后，确保你重启了 Nginx，以使配置生效。

注意事项：
- 在使用 GeoIP2 模块之前，请确保你已经获取了合适的 GeoIP2 数据库文件，并将路径正确配置到 Nginx 中。
- GeoIP2 数据库文件通常需要定期更新，以确保地理位置信息的准确性。
- 当使用 GeoIP2 模块时，要谨慎处理客户端IP地址的隐私问题，尤其是在涉及用户数据的情况下。

使用 GeoIP2 模块可以增强你的网站或应用程序的功能，使你能够更好地定制内容、执行地理位置相关的策略和分析用户行为。



### 15）反向代理缓存proxy_cache

#### 1.介绍

反向代理缓存是一种用于提高网站性能和降低服务器负载的技术。它通过在代理服务器上存储响应数据，以便在后续请求中快速提供相同的数据，从而减少了对目标服务器的请求和数据传输次数。以下是反向代理缓存的主要特点和用途：

1. 提高性能：反向代理缓存允许代理服务器在第一次请求后将响应数据存储在本地。当后续的请求相同的资源时，代理服务器可以立即提供缓存的响应，从而显著减少了响应时间，提高了用户体验。

2. 减少服务器负载：通过将请求缓存，减少了对目标服务器的请求次数，降低了服务器的负载。这对于处理高流量网站或应用程序非常有用，因为它可以减轻服务器的压力。

3. 节省带宽：反向代理缓存可以减少对网络带宽的需求，因为它会将数据存储在代理服务器上，从而减少了数据传输到目标服务器的频率。

4. 动态内容缓存：虽然最初用于静态资源（如图像、CSS文件和JavaScript文件）的缓存，但反向代理缓存也可以用于缓存动态生成的内容，如网页内容和API响应。这有助于加速动态网页的加载速度。

5. 配置和管理：代理服务器通常提供丰富的配置选项，允许管理员控制缓存策略，如缓存时间、不活动时间、缓存清理策略等。

6. 适用于内容分发：反向代理缓存经常用于内容分发网络（CDN），以加速内容的传输和分发到全球各地的用户。

7. 隐私和安全性：需要小心处理缓存的敏感信息，以确保用户数据的隐私和安全性。一些敏感信息可能不适合缓存。

总的来说，反向代理缓存是一种强大的技术，可用于加速网站、降低服务器负载、节省带宽和提高用户体验。它对于大多数Web应用程序都具有重要的性能和效益优势。



#### 2.配置

```nginx
proxy_cache_path /ngx_tmp levels=1:2 keys_zone=test_cache:100m inactive=1d max_size=10g; #

server {
  listen       80;
  server_name  www.vankykoo.cn;

  location / {
    proxy_pass http://leos;
    root   html;
    add_header Nginx_Cache "$upstream_cache_status"; #
    proxy_cache test_cache; #
    proxy_cache_valid 1h; #
  }

  error_page   500 502 503 504  /50x.html;
  location = /50x.html {
    root   html;
  }

}
```

> `proxy_cache_path /ngx_tmp levels=1:2 keys_zone=test_cache:100m inactive=1d max_size=10g;`
>
> 这是一个配置Nginx反向代理服务器中缓存（caching）的指令。让我为你解释这个指令的各个部分：
>
> 1. `proxy_cache_path`: 这是一个Nginx指令，用于定义代理缓存的路径和一些相关配置。它告诉Nginx在哪里存储缓存数据以及如何管理缓存。
>
> 2. `/ngx_tmp`: 这是缓存数据的实际存储路径。在这个示例中，缓存数据将存储在`/ngx_tmp`目录下。你可以根据需要指定不同的目录路径。
>
> 3. `levels=1:2`: 这部分定义了存储缓存数据的子目录结构。这个参数指定了目录层次的结构，以便更高效地存储和检索缓存数据。在这里，`levels=1:2`表示有两个子目录级别。例如，缓存数据可能存储在`/ngx_tmp/3/78`这样的路径中，其中`3`和`78`是子目录。
>
> 4. `keys_zone=test_cache:100m`: 这部分定义了缓存的名称和相关的配置。`test_cache`是缓存名称，用于标识这个缓存区。`100m`表示分配给缓存键的内存大小。缓存键用于存储与缓存相关的信息，如URL和响应头。
>
> 5. `inactive=1d`: 这个参数定义了缓存数据的不活动时间。`1d`表示缓存数据在一天内没有被访问时将被认为是不活动的，因此可以被清除。你可以根据需要调整这个值。
>
> 6. `max_size=10g`: 这个参数定义了缓存的最大大小。在这个示例中，缓存的最大大小被设置为10GB。一旦缓存达到这个大小限制，Nginx将根据LRU（最近最少使用）算法来删除较早的缓存数据。
>
> 这个配置示例设置了一个Nginx代理缓存，用于存储从后端服务器获取的响应数据，以便提高性能和降低服务器负载。缓存路径、层次结构、缓存键的内存分配、不活动时间和最大大小等参数可以根据你的需求进行调整。



#### 3.缓存清理

Nginx的purge模块（ngx_cache_purge模块）是一个用于清理代理服务器缓存的扩展模块。该模块允许你通过发送特定请求来删除Nginx代理缓存中的特定内容。这在需要及时更新缓存内容或清除旧缓存时非常有用。

以下是如何使用nginx的purge模块进行缓存清理的基本步骤：

1. 安装purge模块：首先，你需要确保Nginx已编译并加载了ngx_cache_purge模块【安装地址：https://github.com/FRiCKLE/ngx_cache_purge】。你可以在编译Nginx时添加该模块，或者使用预编译的Nginx版本，它包含了purge模块。

2. 配置Nginx：在Nginx配置文件中进行相应的设置。通常，你需要在`location`部分添加以下配置：

   ```nginx
   location ~ /purge(/.*) {
       allow 127.0.0.1; # 允许本地请求
       deny all;      # 禁止其他IP的请求
       proxy_cache_purge CACHE_NAME $1; # 清除缓存
   }
   ```

   - `/purge`是用于清除缓存的URL路径。
   - `allow 127.0.0.1`允许只有本地请求来访问清除缓存的URL路径。你可以根据需要更改允许的IP地址。
   - `deny all`禁止其他IP地址的请求。
   - `proxy_cache_purge`用于实际的缓存清理操作，其中`CACHE_NAME`应替换为你要清理的缓存区名称。

3. 发送清理请求：要清除缓存，请发送HTTP请求到`/purge`路径，例如：

   ```
   curl -X PURGE http://example.com/purge/url/to/clear
   ```

   这将清除代理服务器缓存中`http://example.com/url/to/clear`的内容。

请注意以下几点：

- 使用purge模块时，要非常小心谨慎，确保你只允许授权的请求来清除缓存，以防止滥用。
- 清除缓存可能会导致服务器负载增加，因为之后的请求需要重新生成缓存。
- 你可以在Nginx的配置文件中定义多个`location`块来处理不同的缓存区域。

使用nginx的purge模块允许你以程序化和迅速的方式清除缓存，从而保持缓存内容的新鲜度，并确保用户获取最新的数据。



#### 4.断点续传

Nginx支持断点续传（Byte Range）功能，允许客户端在下载文件时暂停并在稍后的时间恢复下载，而无需重新下载整个文件。这对于大文件的下载和网络连接不稳定的情况非常有用。以下是有关Nginx断点续传的一些关键信息：

1. HTTP Range 请求头：客户端通过在HTTP请求头中包含Range头来请求特定字节范围的文件数据。例如，Range: bytes=500-999 表示客户端只需下载文件的第501到1000字节。这使得客户端能够请求文件的特定部分而不是整个文件。

2. Nginx的配置：为了启用断点续传功能，您需要在Nginx的配置文件中进行设置。通常，您需要在`location`部分添加以下指令：

```nginx
location /path/to/resource {
    root /your/website/root;
    try_files $uri =404;
    if_range $http_range;
    proxy_set_header Range $http_range;
    proxy_set_header If-Range $http_if_range;
    proxy_pass http://backend;
}
```

这将允许Nginx接收并传递Range请求头，以及If-Range请求头，以便在多次请求相同文件时确保数据的一致性。

3. 静态文件和代理：上述配置适用于静态文件和反向代理情况。如果您想在Nginx上提供静态文件的断点续传支持，只需将`proxy_pass`替换为`alias`或`root`指令，具体取决于您的文件存储方式。如果您使用Nginx作为反向代理服务器，您需要配置Nginx将Range请求传递给后端服务器，以使后端服务器能够处理这些请求。

4. 客户端支持：虽然Nginx提供了断点续传的支持，但客户端应用程序也需要支持Range请求头以及处理部分文件的下载。大多数现代的Web浏览器和下载工具都支持这一特性，因此用户可以通过暂停和恢复下载来获得更好的下载体验。

总之，Nginx的断点续传功能允许客户端在下载文件时更加灵活，而无需重新下载整个文件。这对于减少网络流量和提高下载效率非常有用，特别是在处理大文件或不稳定的网络连接时。





#### 5.proxy_cache相关参数

Nginx中的`proxy_cache`模块允许您配置用于缓存反向代理响应的行为。以下是一些与`proxy_cache`相关的常见可配置参数：

1. `proxy_cache`：用于定义一个缓存块的主要指令，通常用于指定一个缓存区块的名称和一些默认属性。

   示例：
   ```
   proxy_cache my_cache;
   ```

2. `proxy_cache_path`：定义Nginx缓存的物理位置和属性，包括路径、缓存大小、缓存键和其他选项。

   示例：
   ```
   proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=100m;
   ```

   - `levels`：指定缓存目录的层次结构。
   - `keys_zone`：定义缓存的共享内存区名称和大小。
   - `max_size`：指定缓存的最大大小。

3. `proxy_cache_valid`：指定响应的缓存时间，以便后续请求可以重复使用缓存的响应。

   示例：
   ```
   proxy_cache_valid 200 302 10m;
   ```

   这表示HTTP状态码200和302的响应将在缓存中保留10分钟。

4. `proxy_cache_key`：定义如何生成缓存键，通常使用变量来创建一个唯一的缓存标识符。

   示例：
   ```
   proxy_cache_key "$host$request_uri$is_args$args";
   ```

   这将以主机名、请求URI和查询参数作为缓存键。

5. `proxy_cache_bypass`：允许您指定一些条件，当满足时，不会使用缓存的响应，而是直接请求后端服务器。

   示例：
   ```
   proxy_cache_bypass $cookie_nocache;
   ```

   这将根据名为`nocache`的Cookie的存在与否来决定是否绕过缓存。

6. `proxy_cache_use_stale`：定义当后端服务器无法提供响应时，是否使用缓存的内容。

   示例：
   ```
   proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
   ```

   这将在后端服务器返回错误、超时或正在更新缓存时使用缓存。

7. `proxy_cache_methods`：指定哪些HTTP请求方法的响应应该被缓存。

   示例：
   ```
   proxy_cache_methods GET HEAD;
   ```

   这将只缓存GET和HEAD请求的响应。

这些是一些与`proxy_cache`相关的常见配置指令和参数。根据您的需求和使用场景，您可以灵活配置这些参数以满足您的反向代理缓存要求。



> 在实际项目中，`proxy_cache`的配置样例可以根据特定需求和场景而异。以下是一个示例，演示了如何配置Nginx的`proxy_cache`来缓存反向代理的响应：
>
> ```nginx
> http {
>     proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=100m;
>     
>     server {
>         listen 80;
>         server_name example.com;
>
>         location / {
>             proxy_pass http://backend_server;
>             
>             proxy_cache my_cache;  # 使用名为my_cache的缓存块
>             proxy_cache_valid 200 302 10m;  # 缓存HTTP状态码200和302的响应10分钟
>             proxy_cache_valid 404 1m;  # 缓存HTTP状态码404的响应1分钟
>             
>             proxy_cache_key "$host$request_uri$is_args$args";
>             proxy_cache_bypass $cookie_nocache;  # 根据Cookie值绕过缓存
>             proxy_no_cache $arg_nocache;  # 根据查询参数绕过缓存
>         }
>     }
> }
> ```
>
> 这是一个简单的Nginx配置示例，用于缓存`example.com`网站的反向代理响应。这个配置包括以下要点：
>
> - `proxy_cache_path`指令定义了缓存的物理位置和属性，如路径、缓存大小、缓存键和其他选项。
>
> - `proxy_cache`指令在`location`部分启用了缓存，并使用名为`my_cache`的缓存块。
>
> - `proxy_cache_valid`指令配置了响应的缓存时间，这里缓存HTTP状态码200和302的响应10分钟，404的响应1分钟。
>
> - `proxy_cache_key`指令定义了如何生成缓存键，基于主机名、请求URI和查询参数。
>
> - `proxy_cache_bypass`和`proxy_no_cache`指令允许根据Cookie值和查询参数绕过缓存。
>
> 这个示例只涵盖了一些基本的`proxy_cache`配置选项。根据您的具体需求，您可以进一步调整这些选项以满足您的项目要求。在实际项目中，您还可以根据不同的`location`块、后端服务器和缓存策略来配置`proxy_cache`。



### 16）多级缓存总结

![](C:\Users\86180\Desktop\picPick\238.png)



## 二、高效

###1）sendfile

`sendfile` 是 Nginx 服务器中的一个特性，用于在发送静态文件时提高性能。它是通过操作系统提供的零拷贝（Zero-Copy）技术来实现的。`sendfile` 允许 Nginx 直接在文件和网络套接字之间传输数据，而不必将数据从文件复制到应用程序缓冲区，然后再传输到网络套接字。这可以减少数据复制的开销，提高数据传输的效率。

以下是 `sendfile` 的主要特点和用法：

1. **零拷贝技术：** `sendfile` 利用操作系统提供的零拷贝技术，**将文件数据直接传输到网络套接字，而无需在用户空间和内核空间之间进行额外的数据复制。**这减少了CPU和内存开销，提高了性能。

   > 当我们想要把一个文件的内容传输到另一个地方时，通常的方法是先把文件的内容从磁盘复制到计算机的内存，然后再从内存传输到目标地点。这个过程中，数据需要被复制两次，一次从磁盘到内存，再一次从内存到目标地点。
   >
   > 而零拷贝则是一种更高效的方式。它避免了数据被复制两次的过程。具体来说，当使用零拷贝时，数据会直接从磁盘传输到目标地点，而不需要首先复制到内存。这就好像你把文件从一个地方搬到另一个地方，而不需要在中间存放在你的手里。
   >
   > 零拷贝的好处在于它减少了数据复制的开销，因为数据不需要在内存中暂时存储，从而减少了计算机的 CPU 和内存使用，提高了数据传输的效率。这对于大文件的传输和高性能应用程序非常有用，因为它可以减少系统资源的开销，加速数据传输过程。

2. **静态文件传输：** `sendfile` 通常用于传输静态文件，如图片、视频、CSS、JavaScript 等，因为这些文件通常很大，且可以受益于零拷贝技术。

3. **Nginx 配置：** 在 Nginx 配置中，`sendfile` 可以通过 `sendfile on;` 或 `sendfile off;` 来启用或禁用。默认情况下，`sendfile` 是启用的。

   ```nginx
   http {
       server {
           location /static/ {
               sendfile on;
               root /path/to/static/files;
           }
       }
   }
   ```

4. **适用于大文件：** `sendfile` 尤其适用于大文件的传输，因为它能够显著减少数据复制和处理的开销。

5. **性能优势：** 使用 `sendfile` 可以提高 Nginx 服务器的性能，降低对 CPU 和内存的负荷，从而更有效地处理大量的并发请求。

需要注意的是，`sendfile` 只适用于静态文件的传输。对于动态内容，如通过后端应用程序生成的响应，`sendfile` 不起作用，因为这些内容需要在应用程序内部生成并发送。因此，在配置 Nginx 时，应根据文件类型和应用程序的需求合理使用 `sendfile` 特性。



#### 配合open_file_cache

Nginx的`open_file_cache`是用于管理打开文件的缓存，旨在提高服务器性能和降低磁盘I/O负载。当Nginx服务器处理请求时，通常需要打开和关闭文件来提供静态资源（如HTML、CSS、JavaScript、图像等）。如果频繁地进行这些文件操作，会导致较大的性能开销。`open_file_cache`允许Nginx将文件描述符缓存到内存中，以避免重复的打开和关闭操作。

以下是`open_file_cache`的主要特点和相关配置选项：

1. `open_file_cache`主要特点：
   - 加速性能：通过避免频繁的文件打开和关闭操作，Nginx可以显著提高服务器性能，减少磁盘I/O操作，降低延迟。
   - 减轻系统负担：避免不必要的系统调用和文件操作，有助于降低对操作系统资源的占用。
   - 避免文件描述符泄漏：Nginx会负责管理文件描述符的打开和关闭，以防止文件描述符泄漏问题。

2. `open_file_cache`相关配置选项：

   - `open_file_cache`: 启用或配置文件缓存的主要指令。可以在`http`、`server`、或`location`部分进行配置。示例：

     ```nginx
     http {
         open_file_cache max=1000 inactive=20s;
     }
     ```

     - `max`: 指定缓存中可以存储的文件描述符的最大数量。
     - `inactive`: 指定文件在缓存中保持活动状态的时间。过了这个时间，文件描述符将被关闭。

   - `open_file_cache_valid`: 配置了缓存中文件的有效期，指定文件缓存有效的时间。如果文件的访问时间超过此时间，将被认为是过期的文件。

     ```nginx
     open_file_cache_valid 30s;
     ```

     这表示文件在缓存中保持有效30秒。

   - `open_file_cache_min_uses`: 指定文件在缓存中保持的最小使用次数。如果一个文件被使用次数不足，它可能会被移出缓存。

     ```nginx
     open_file_cache_min_uses 2;
     ```

     这表示文件必须至少被使用2次，才会被保留在缓存中。

   - `open_file_cache_errors`: 指定错误代码，如果发生指定的错误，相关文件的描述符将被关闭。

     ```nginx
     open_file_cache_errors 404 500 502;
     ```

     这表示当发生HTTP状态码404、500或502的错误时，相关文件描述符将被关闭。

   - `open_file_cache_lock`: 指定是否应启用文件锁定来避免竞态条件。

     ```nginx
     open_file_cache_lock on;
     ```

     启用文件锁定以防止多个进程同时打开相同的文件。

通过配置`open_file_cache`，Nginx可以更高效地处理静态文件，降低了服务器的负载，并提高了性能。这对于高流量的网站以及需要频繁访问大量静态资源的应用程序非常有用。





### 2）外置缓存

#### 1.error_page

`error_page` 是 Nginx 配置中用于定义错误页面的指令。它允许您指定在服务器遇到特定 HTTP 错误代码时应显示的自定义错误页面，以提供更好的用户体验和友好的错误信息。以下是有关 `error_page` 的一些重要信息：

1. **基本语法**：

   ```nginx
   error_page error_code [URI];
   ```

   - `error_code` 是 HTTP 错误代码，例如 404 表示页面未找到，500 表示服务器内部错误。
   - `[URI]` 是可选的自定义错误页面的路径或 URL。您可以将其设置为指向您自定义的错误页面的文件路径或外部 URL。

2. **示例**：

   ```nginx
   error_page 404 /custom_404.html;
   error_page 404 =302 http://www.google.com;
   error_page 500 502 503 504 /custom_error.html;
   ```

   在这个示例中，当服务器返回 HTTP 404 错误时，Nginx 会显示自定义的 `/custom_404.html` 页面。类似地，当服务器返回 HTTP 500、502、503 或 504 错误时，将显示自定义的 `/custom_error.html` 页面。

3. **内置错误页面**：

   Nginx还提供了一些默认的内置错误页面，这些页面位于 Nginx 安装目录的 `html` 子目录下。您可以使用这些内置错误页面，或者创建自己的自定义页面，并在 `error_page` 指令中指定它们的路径。

4. **错误页面的设计和内容**：

   自定义错误页面通常包含有关错误的说明、解决方法或其他相关信息，以帮助用户理解发生的问题。它们可以包含 HTML 标记、样式和图像，以使页面更具吸引力和信息丰富。

5. **多级错误页面**：

   您可以为不同的 HTTP 错误代码设置不同的自定义错误页面。如果没有明确设置某个错误代码的自定义页面，Nginx将尝试查找更一般的错误页面（例如，404 页面对应于 "Not Found" 错误，500 页面对应于服务器错误等）。

6. **error_page 指令的位置**：

   `error_page` 指令通常位于 `http`、`server` 或 `location` 部分中，以覆盖特定范围的错误页面。例如，您可以在 `server` 部分中定义一个通用的错误页面，然后在 `location` 部分中定义特定的错误页面。

通过使用 `error_page`，您可以为不同类型的错误提供定制化的用户体验，以帮助用户更好地理解和处理错误情况。这对于提高网站的可用性和用户满意度非常重要。



####2.命名location

在 Nginx 配置中，命名 location 是一个具有名称的 `location` 块，通常用于处理特定类型的请求或执行自定义逻辑。命名 location 允许您为特定的请求处理逻辑分配一个易于理解的名称，以提高配置的可读性和可维护性。

以下是如何定义和使用命名 location 的基本示例：

```nginx
location @my_named_location {
    # 这是一个命名 location，名称为 @my_named_location
    # 在这里可以定义处理特定请求的逻辑
}
```

要在服务器块中使用命名 location，通常需要遵循以下步骤：

1. 定义命名 location：在 Nginx 配置文件的任何地方，您可以使用 `location` 指令创建一个命名 location，并为其指定一个名称。例如，`@my_named_location` 是一个命名 location。

2. 在其他 location 或 server 部分中引用命名 location：您可以在其他 `location` 或 `server` 部分中使用命名 location，以触发其中定义的逻辑。通常使用 `try_files` 或 `error_page` 等指令内部重定向到命名 location。

下面是一个示例，展示了如何定义和使用命名 location 来处理特定类型的请求：

```nginx
location @my_named_location {
    # 这是一个命名 location，用于处理特定类型的请求
    return 200 "This is my named location.";
}

server {
    listen 80;
    server_name example.com;

    location /special {
        # 当请求以 /special 开头时，内部重定向到 @my_named_location
        try_files $uri @my_named_location;
    }

    location / {
        # 默认 location，处理其他请求
    }
}
```

在这个示例中，`location @my_named_location` 定义了一个命名 location，当它被内部重定向时，会返回一条自定义消息。然后，在服务器块中，我们使用 `location /special` 来处理以 `/special` 开头的请求，并通过 `try_files` 指令将这些请求内部重定向到 `@my_named_location`，以触发命名 location 的处理逻辑。

这样，您可以为不同类型的请求定义命名 location，以使配置更具结构和可读性，同时可以在不同的位置内部重定向到这些命名 location 来处理请求。



#### 3.nginx+memcached

![](C:\Users\86180\Desktop\picPick\239.png)

```nginx
server {
        listen       80;
        server_name  www.vankykoo.cn;

        location / {

	    	set $memcached_key "$uri?$args";
            memcached_pass 127.0.0.1:11211;

	    	add_header X-Cache-Satus HIT;

	    	add_header Content-Type 'text/html; charset=utf-8';
        }
	
	
 	error_page 404 = @diy_location;
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

	location @diy_location{
	    proxy_set_header memcached_key $memcached_key;
	    proxy_pass http://192.168.200.140:8080;
	}
```



#### 4.nginx+redis

```nginx
	location /foo {
	    default_type text/html;
	    set $value "first";
	    redis2_query set one $value;
	    redis2_pass 127.0.0.1:6379;
	}

	location /get {
		default_type text/html;
     		redis2_pass 127.0.0.1:6379;
     		#set_unescape_uri $key $arg_key;  # this requires ngx_set_misc
     		redis2_query get $arg_key;
	}
```



### 3）MySQL集群

```nginx
stream {

    upstream mysql{
		server 192.168.200.140:3306; # 主
		server 192.168.200.142:3306; # 从
    }

    server{
		listen 3306;
		proxy_pass mysql;
    }

}
```



###4）QPS限制

在Nginx中，有两种主要的限流方式：

1. **单位时间访问限流QPS**：使用Nginx的`ngx_http_limit_req_module`模块进行限制，使用`limit_req`和`limit_req_zone`命令来设置。例如：

```nginx
# http模块下添加
limit_req_zone $binary_remote_addr zone=ip_limit:10m rate=10r/s;

# location模块下添加
limit_req zone=ip_limit burst=15 nodelay;
```

这里的`limit_req_zone`命令定义了限速的参数，`$binary_remote_addr`是基于`remote_addr`（客户端IP）来做限流，`ip_limit:10m`表示一个大小为10M，名字为`ip_limit`的内存区域，`rate=10r/s`表示每秒最多处理10个请求。`limit_req`命令在所在的location使能定义的速率。

2. **限制并发连接数**：使用Nginx的`ngx_http_limit_conn_module`模块进行限制，涉及命令`limit_conn_zone` 和 `limit_conn`。例如：

```nginx
limit_conn perip 10;  # 对应的key是 $binary_remote_addr，表示限制单个IP同时最多能持有10个连接。
limit_conn perserver 1000;  # 对应的key是 $server_name，表示虚拟主机 (server) 同时能处理并发连接的总数，即该服务能承受的最大并发数。
```



```nginx
# 这里表示1s内只允许有50个请求打入
limit_req_zone $binary_remote_addr zone=one:10m rate=50r/s; # http块中
limit_req zone=one; # server块的location中
```



###5）限制带宽传输（传输速度）

在Nginx中，可以通过`limit_rate`指令来限制连接的带宽。这个指令可以在`http`，`server`，`location`，和`if in location`等级别中使用。

例如，如果你想限制每个连接的带宽为100K/s，你可以在`location`块中设置如下：

```nginx
location /download/ {
    limit_rate 100k;
}

```

另外，你还可以使用`limit_rate_after`指令来设置在发送了多少字节的数据后开始限制带宽。例如：

```nginx
location /download/ {
    limit_rate 100k;
    limit_rate_after 500k;
}
```

这样，当从`/download/`目录下载文件时，前500K的数据会以最大速度发送，然后剩余的数据会以100K/s的速度发送。



### 6）限制并发数

在Nginx中，可以通过`ngx_http_limit_conn_module`模块来限制并发连接数。这个模块提供了两个主要的指令：`limit_conn_zone`和`limit_conn`。

1. `limit_conn_zone`：这个指令用于定义限制的参数。例如，`limit_conn_zone $binary_remote_addr zone=addr:10m;`这个指令定义了一个名为`addr`的共享内存区域，大小为10M，用于存储每个IP地址的并发连接数。
2. `limit_conn`：这个指令用于在特定的位置启用定义的限制。例如，`limit_conn addr 10;`这个指令表示，对于每个IP地址，同时最多能持有10个连接。





### 7）日志

在Nginx中，主要有两种类型的日志：访问日志和错误日志。以下是这两种日志的主要配置参数：

1. 访问日志（access_log）：访问日志主要记录客户端的请求。其基本语法和参数如下：

   ```nginx
   access_log path [format [buffer=size] [gzip [=level]] [flush=time] [if=condition]];
   ```

- path：指定日志的存放位置。
- format：指定日志的格式，默认使用预定义的combined。
- buffer：用来指定日志写入时的缓存大小，默认是64k。
- gzip：日志写入前先进行压缩，压缩率可以指定，从1到9数值越大压缩比越高，同时压缩的速度也越慢，默认是1。
- flush：设置缓存的有效时间，如果超过flush指定的时间，缓存中的内容将被清空。
- if：条件判断，如果指定的条件计算为0或空字符串，那么该请求不会写入日志。

2. 错误日志（error_log）：错误日志记录了访问出错的信息，可以帮助我们定位错误的原因。

此外，你还可以使用log_format指令来自定义日志格式。例如：

```nginx
log_format name [escape=default|json] string ...;
```

- name：格式名称，在access_log指令中引用。
- escape：设置变量中的字符编码方式是json还是default，默认是default。
- string：要定义的日志格式内容，该参数可以有多个，参数中可以使用Nginx变量。




### 8）主动检查上游健康状态

#### 1.重试机制

```nginx
# 可以在upstream中配置
upstream leos {
	server 192.168.200.140:8080 max_fails=5 fail_timeout=10s;
	server 192.168.200.142:8080;
}

# 可以在location中配置
location / {
	proxy_next_upstream error timeout;
  	proxy_next_upstream_timeout 15s;
  	proxy_next_upstream_tries 5;
  	proxy_pass http://leos;

 	root html;
}
```

**max_fails** ：最大失败次数，0为标记一直可用，不检查健康状态

**fail_timeout** ：失败时间，当fail_timeout时间内失败了max_fails次，标记服务不可用，fail_timeout时间后会再次激活次服务

**proxy_next_upstream** 

**proxy_next_upstream_timeout** ：重试最大超时时间

**proxy_next_upstream_tries** ：重试次数，包括第一次proxy_next_upstream_timeout时间内允许proxy_next_upstream_tries次重试



#### 2.tengine健康检查

①到https://github.com/yaoweibin/nginx_upstream_check_module下载模块

②复制patch，用了1.20的nginx。

③在linux中 `vim /root/path`，编辑并把patch粘贴进去。

④使用 `patch -p1 < /root/path` 

⑤编译安装nginx20.

⑥配置

```nginx
#location
location /status {
  	check_status;
    access_log off;
}

location / {
  	proxy_pass http://backend;
  	root   html;
}

# upstream
upstream backend {
   	server 192.168.200.140:8080;
   	server 192.168.200.142:8080;
   	check interval=3000 rise=2 fall=5 timeout=1000 type=http;
 	check_http_send "HEAD / HTTP/1.0\r\n\r\n";
 	check_http_expect_alive http_2xx http_3xx;
}
```

⑦访问/status就可以进入查看健康状态的页面



### 9）Lua语法

####1.while循环

```lua
local i = 0

local max = 10

while i <= max do

print(i)

i = i +1

end
```



####2.if-else

```lua
local function main()


local age = 140

local sex = 'Male'
 

  if age == 40 and sex =="Male" then
    print(" 男人四十一枝花 ")
  elseif age > 60 and sex ~="Female" then
   
    print("old man!!")
  elseif age < 20 then
    io.write("too young, too simple!\n")
  
  else
  print("Your age is "..age)
  end

end


-- 调用
main()

```



####3.for循环

```lua
sum = 0

for i = 100, 1, -2 do

	sum = sum + i

end
```



####4.函数

```lua
function myPower(x,y)

  return      y+x

end

power2 = myPower(2,3)

 print(power2)
```



```lua
function newCounter()

   local i = 0
   return function()     -- anonymous function

        i = i + 1

        return i

    end
end

 

c1 = newCounter()

print(c1())  --> 1

print(c1())  --> 2

print(c1())
```



####5.返回值

```lua
name, age,bGay = "yiming", 37, false, "yimingl@hotmail.com"

print(name,age,bGay)
```



```lua
function isMyGirl(name)
  return name == 'xiao6' , name
end

local bol,name = isMyGirl('xiao6')

print(name,bol)
```



####6.Table

key，value的键值对 类似 map

```lua
local function main()
dog = {name='111',age=18,height=165.5}

dog.age=35

print(dog.name,dog.age,dog.height)

print(dog)
end
main()

```



####7.数组

```lua
local function main()
arr = {"string", 100, "dog",function() print("wangwang!") return 1 end}

print(arr[4]())
end
main()

```



####8.遍历

```lua
arr = {"string", 100, "dog",function() print("wangwang!") return 1 end}

for k, v in pairs(arr) do

   print(k, v)
end
```



####9.成员函数

```lua
local function main()

person = {name='旺财',age = 18}

  function  person.eat(food)

    print(person.name .." eating "..food)

  end
person.eat("骨头")
end
main()
```



### 10）openresty

OpenResty是一个基于Nginx的Web开发平台，它通过集成一系列功能丰富的第三方模块来扩展Nginx的能力。OpenResty的核心组件是LuaJIT，它为Nginx提供了强大的脚本编程能力，使得开发人员可以使用Lua语言编写高效、灵活的Web应用。

通过使用OpenResty，开发人员可以通过Lua脚本动态地处理HTTP请求和响应，实现诸如URL重写、访问控制、缓存加速、负载均衡等功能。此外，OpenResty还提供了许多内置模块，例如Redis、MySQL等，使得与这些常见的数据存储和处理系统的交互变得简单。

OpenResty的优势在于其高性能和可扩展性。得益于Nginx的事件驱动架构和OpenResty的LuaJIT引擎，它具有出色的并发处理能力和低资源消耗。此外，OpenResty可以通过加载不同的模块来满足不同的需求，可以轻松扩展其功能，使得开发人员可以根据项目需要选择合适的模块进行集成。

总而言之，OpenResty提供了一种快速、高效的方式来开发和扩展Web应用程序。它结合了Nginx的性能优势和Lua的灵活性，为开发人员提供了一个强大的工具来构建高性能、可扩展的Web服务。





####1.配置

启动openresty：`./nginx -c /usr/local/openrest/nginx/conf/nginx.conf`

关闭：`./nginx -s stop`	这个nginx是openresty/nginx/sbin/nginx



①直接在配置文件中写lua脚本，不推荐。

```nginx
location /lua {
 	default_type text/html;
 	 	content_by_lua '
 		ngx.say("<p>Hello, World!</p>")
 	';
}
```

②引用lua脚本

相对路径是openresty/nginx/

```nginx
location /lua {
 	default_type text/html;
 	content_by_lua_file Lua/hello.lua;
}
```



####2.lua脚本热部署

在http块中加入配置：

```nginx
lua_code_cache off;
```



#### 3.获取请求信息的lua脚本

①获取Nginx请求头信息

```lua
local headers = ngx.req.get_headers()                         

ngx.say("Host : ", headers["Host"], "<br/>")  

ngx.say("user-agent : ", headers["user-agent"], "<br/>")  

ngx.say("user-agent : ", headers.user_agent, "<br/>")

for k,v in pairs(headers) do  

    if type(v) == "table" then  

        ngx.say(k, " : ", table.concat(v, ","), "<br/>")  

    else  

        ngx.say(k, " : ", v, "<br/>")  

    end  

end  
```



②获取post请求参数

```lua
ngx.req.read_body()  

ngx.say("post args begin", "<br/>")  

local post_args = ngx.req.get_post_args()  

for k, v in pairs(post_args) do  

    if type(v) == "table" then  

        ngx.say(k, " : ", table.concat(v, ", "), "<br/>")  

    else  

        ngx.say(k, ": ", v, "<br/>")  

    end  
end
```



③http协议版本

```lua
ngx.say("ngx.req.http_version : ", ngx.req.http_version(), "<br/>")
```



④请求方法

```lua
ngx.say("ngx.req.get_method : ", ngx.req.get_method(), "<br/>")  
```



⑤原始的请求头内容

```lua
ngx.say("ngx.req.raw_header : ",  ngx.req.raw_header(), "<br/>")  
```



⑥body内容体

```lua
ngx.say("ngx.req.get_body_data() : ", ngx.req.get_body_data(), "<br/>")
```

 

#### 4.nginx缓存

①nginx全局内存缓存

在配置文件http块中配置

```nginx
lua_shared_dict shared_data 1m;
```



lua脚本实例

```lua
local shared_data = ngx.shared.shared_data  

local i = shared_data:get("i")  

if not i then  

    i = 1  

    shared_data:set("i", i)  

    ngx.say("lazy set i ", i, "<br/>")  
end  
 
i = shared_data:incr("i", 1)  

ngx.say("i=", i, "<br/>")
```



②lru

配置：

在location中配置：

在lualib/my/找cache.lua脚本

```nginx
content_by_lua_block {
  	require("my/cache").go()
}
```



#### 5.连接redis



#### 6.连接mysql

在 OpenResty 中连接 MySQL 数据库通常涉及使用第三方模块，最常用的是 lua-resty-mysql。下面是一个简单的介绍：

lua-resty-mysql 是 OpenResty 生态系统中用于连接 MySQL 数据库的模块，它提供了与 MySQL 数据库进行交互的能力，并且充分利用了 Nginx 的事件模型和非阻塞 I/O 特性，使得数据库访问不会阻塞整个 Nginx 服务器的运行。

连接 MySQL 数据库的基本步骤如下：

1. 首先需要通过 Lua 的 require 函数引入 lua-resty-mysql 模块：

   ```lua
   local mysql = require "resty.mysql"
   ```

2. 接着创建一个 MySQL 客户端实例：

   ```lua
   local client, err = mysql:new()
   if not client then
       ngx.log(ngx.ERR, "failed to instantiate mysql: ", err)
       return
   end
   ```

3. 设置连接信息，如 MySQL 服务器的地址、端口、用户名、密码等：

   ```lua
   local ok, err, errno, sqlstate = client:connect{
       host = "mysql_host",
       port = mysql_port,
       database = "my_db",
       user = "my_user",
       password = "my_password",
       max_packet_size = 1024 * 1024,
   }
   if not ok then
       ngx.log(ngx.ERR, "failed to connect: ", err, ": ", errno, " ", sqlstate)
       return
   end
   ```

4. 执行 SQL 查询或更新操作：

   ```lua
   local res, err, errno, sqlstate = client:query("select * from my_table")
   if not res then
       ngx.log(ngx.ERR, "bad result: ", err, ": ", errno, ": ", sqlstate, ".")
       return
   end
   ```

5. 最后，务必在使用完成后关闭连接：

   ```lua
   local ok, err = client:close()
   if not ok then
       ngx.log(ngx.ERR, "failed to close: ", err)
       return
   end
   ```

需要注意的是，由于 OpenResty 是基于 Nginx 构建的，其事件驱动的特性使得 lua-resty-mysql 可以实现高并发的数据库访问。然而，在实际使用中，还需要考虑连接池管理、异常处理、性能优化等问题，以确保数据库访问的安全可靠和高效稳定。



#### 7.模板引擎

配置：

```nginx
location /lua {
  default_type text/html;
  set $template_root /usr/local/openrest/tpl;
  content_by_lua_file Lua/tpl.lua; # 相对路径在/usr/local/openrest/nginx下
}
```



lua脚本

```lua
-- Using template.new
-- /usr/local/openrest/nginx/Lua/tpl.lua
local template = require "resty.template"
local view = template.new "view.html"
view.message = "Hello, World!"
view:render()
```



openresty目录下创建tpl模板目录，里面放模板文件

```html
<!DOCTYPE html>
<html>
<body>
  <h1>{{message}}</h1>
</body>
</html>
```



还需要准备lua-resty-template-2.0模块，将template有关的文件和文件夹移到/usr/local/openrest/lualib/resty之下


















































































































