This paper describes the site isolation architecture used in Chrome, some challenges of it's implementation and it's limitations.

Site isolation is used to defend the following two kinds of attackers:

(1) Renderer exploit attackers who can discover and exploit vulnerabilities to bypass security checks or **even achieve arbitrary code execution in the renderer process.**

(2) Memory disclosure attackers who can not run arbitrary code or lie to the browser process, **but can disclose arbitrary data within a renderer process's address space.** (This can be achieved by transit execution attacks like Spectre and Meltdown)

## Site Isolation Browser Architecture

Site isolation means isolating web pages from different sites in different processes. It's obvious that site isolation can avoid memory disclosure attackers from reading data from other site's process. More details about site isolation are discussed below.

#### Definition of Site

Site refers to the **base domain** of a web page instead of the FQDN. For example, base domain of `https://bar.foo.example.com` is `example.com`. Using site as isolation granularity instead of origin preserves compatibility with up to 13.4% of page loads that change their origin at runtime by assigning `document.domain`. Here compatibility means we don't have to make the JavaScript engine to send a signal when a page assigns it's `document.domain` to switch the page to another renderer process.

#### Cross-Site Iframes

**Basic concept.** Some times a page may embed iframes from different sites. So the main frame and the embedded frame must be rendered in different process. To support such out-of-process frames, the browser maintains a frame tree of each web page, which contains all frames' information and their corresponding render process. Meanwhile, each renderer process also maintains a frame tree for the page (We assume each renderer process only renders a single web page here) it renders. Frames from the site of the renderer process corresponds to a **document node** in the tree while frames from different sites correspond to **proxy nodes**. Communications between two document nodes happen intra-process. But when a document node wants to communicate with a proxy node, it should firstly send the message to the browser process, then browser process will route the message to the process where the real frame of the proxy node is rendered. Note that the actual document and proxy objects live in renderer process; the corresponding browser-side objects are stubs that track state and route IPC messages between the browser and renderer processes.

<img src="Site Isolation.png" style="zoom:50%;" />

<center><b>Figure1.</b> Cross-Process Frames</center>

**State Replication.** Because the state of frames (e.g. the name of frames) are replicated in different places (in the browser process and several renderer processes), when the state is changed (e.g. rename a frame), the message will be sent to the browser process and then broadcast to all affected processes. 

**Painting.** Another problem introduced by cross-site frames is how to combine the frames rendered by different processes to a whole web page. The paper claims the composition happens in a separate process, the granularity is **surface** instead of **frame**. This is an optimization of the browser which aggregates several continuous document node in the frame tree (actually continuous same-site frames in the page) to a **widget** in order to reduce the components. A widget corresponds to surface when painting a page, but it has other usages like input event routing which I can not understand now.

#### Cross-Origin Read Blocking

Site isolation guarantees that a memory disclosure attackers can not access data of documents from different sites, but a document can access resources from different sites via network, here we discuss the following 2 scenarios:

(1) A web page contains the following HTML tag which may contain resource from different sites: `<script>`, `<img>` and `<style>`.

(2) A web page contains a script which uses `XMLHttpRequest` or `fetch` to request resources.

The existing CORS mechanism mainly focuses on the second scenario. Although some HTML tags (e.g. `<img>`, `<script>` and `<link>`) can also contain the `crossorigin` attribute which indicates the request will apply CORS when sending the request[1]. But the premise of a memory disclosure attack is the attacker lures a victim user to visit a elaborated malicious web page, which will not contain the `crossorigin` attribute. So CORS is not applicable to the first scenario. 



### References

[1] https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/crossorigin

