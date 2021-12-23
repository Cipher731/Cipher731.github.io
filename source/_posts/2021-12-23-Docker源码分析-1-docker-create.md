---
title: Docker源码分析 - 1 - docker create
tags:
  - Docker
  - Cloud Native
  - Code Audit
categories: Docker Audit
date: 2021-12-23 15:56:30
---


## 0. 前言
这篇文章将会分析执行`docker create ubuntu`以后，到底发生了哪些事情
## 1. Docker CLI（docker/cli）
最先发生的是解析命令行命令，并且做进一步的处理的操作。
相关的代码位于[docker/cli](https://github.com/docker/cli)仓库，并不在moby/moby项目中。

docker使用了cobra作为CLI的框架，代码设计时`NewXXXCommand`用于注册相关命令
```golang
// cli/command/container/create.go:43

// NewCreateCommand creates a new cobra.Command for `docker create`
func NewCreateCommand(dockerCli command.Cli) *cobra.Command {
	var opts createOptions
	var copts *containerOptions

	cmd := &cobra.Command{
		Use:   "create [OPTIONS] IMAGE [COMMAND] [ARG...]",
		Short: "Create a new container",
		Args:  cli.RequiresMinArgs(1),
		RunE: func(cmd *cobra.Command, args []string) error {
			copts.Image = args[0]
			if len(args) > 1 {
				copts.Args = args[1:]
			}
			return runCreate(dockerCli, cmd.Flags(), &opts, copts)
		},
	}
	// ...
}
```
参数被传入`runCreate`执行，`runXXX`很显然用于执行相关命令
```golang
// cli/command/container/create.go:77

func runCreate(dockerCli command.Cli, flags *pflag.FlagSet, options *createOptions, copts *containerOptions) error {
	proxyConfig := dockerCli.ConfigFile().ParseProxyConfig(dockerCli.Client().DaemonHost(), opts.ConvertKVStringsToMapWithNil(copts.env.GetAll()))
	newEnv := []string{}
	for k, v := range proxyConfig {
		if v == nil {
			newEnv = append(newEnv, k)
		} else {
			newEnv = append(newEnv, fmt.Sprintf("%s=%s", k, *v))
		}
	}
	copts.env = *opts.NewListOptsRef(&newEnv, nil)
	containerConfig, err := parse(flags, copts, dockerCli.ServerInfo().OSType)
	if err != nil {
		reportError(dockerCli.Err(), "create", err.Error(), true)
		return cli.StatusError{StatusCode: 125}
	}
	if err = validateAPIVersion(containerConfig, dockerCli.Client().ClientVersion()); err != nil {
		reportError(dockerCli.Err(), "create", err.Error(), true)
		return cli.StatusError{StatusCode: 125}
	}
	response, err := createContainer(context.Background(), dockerCli, containerConfig, options)
	if err != nil {
		return err
	}
	fmt.Fprintln(dockerCli.Out(), response.ID)
	return nil
}
```
首先是对参数进行了解析，生成了容器的所有配置，其实是属于OCI中runtime-spec规定的config格式。
parse的代码非常多，所以如果有需要再回来看parse代码。
```golang
// cli/command/container/opts.go:303

type containerConfig struct {
	Config           *container.Config
	HostConfig       *container.HostConfig
	NetworkingConfig *networktypes.NetworkingConfig
}
```
完整的containerConfig由Config、HostConfig和NetworkingConfig三者组成
<details>
<summary>containerConfig的组成部分</summary>
```golang
// vendor/github.com/docker/docker/api/types/container/config.go:37

// Config contains the configuration data about a container.
// It should hold only portable information about the container.
// Here, "portable" means "independent from the host we are running on".
// Non-portable information *should* appear in HostConfig.
// All fields added to this struct must be marked `omitempty` to keep getting
// predictable hashes from the old `v1Compatibility` configuration.
type Config struct {
	Hostname        string              // Hostname
	Domainname      string              // Domainname
	User            string              // User that will run the command(s) inside the container, also support user:group
	AttachStdin     bool                // Attach the standard input, makes possible user interaction
	AttachStdout    bool                // Attach the standard output
	AttachStderr    bool                // Attach the standard error
	ExposedPorts    nat.PortSet         `json:",omitempty"` // List of exposed ports
	Tty             bool                // Attach standard streams to a tty, including stdin if it is not closed.
	OpenStdin       bool                // Open stdin
	StdinOnce       bool                // If true, close stdin after the 1 attached client disconnects.
	Env             []string            // List of environment variable to set in the container
	Cmd             strslice.StrSlice   // Command to run when starting the container
	Healthcheck     *HealthConfig       `json:",omitempty"` // Healthcheck describes how to check the container is healthy
	ArgsEscaped     bool                `json:",omitempty"` // True if command is already escaped (meaning treat as a command line) (Windows specific).
	Image           string              // Name of the image as it was passed by the operator (e.g. could be symbolic)
	Volumes         map[string]struct{} // List of volumes (mounts) used for the container
	WorkingDir      string              // Current directory (PWD) in the command will be launched
	Entrypoint      strslice.StrSlice   // Entrypoint to run when starting the container
	NetworkDisabled bool                `json:",omitempty"` // Is network disabled
	MacAddress      string              `json:",omitempty"` // Mac Address of the container
	OnBuild         []string            // ONBUILD metadata that were defined on the image Dockerfile
	Labels          map[string]string   // List of labels set to this container
	StopSignal      string              `json:",omitempty"` // Signal to stop a container
	StopTimeout     *int                `json:",omitempty"` // Timeout (in seconds) to stop a container
	Shell           strslice.StrSlice   `json:",omitempty"` // Shell for shell-form of RUN, CMD, ENTRYPOINT
}
```

```golang
// vendor/github.com/docker/docker/api/types/container/host_config.go:391

// HostConfig the non-portable Config structure of a container.
// Here, "non-portable" means "dependent of the host we are running on".
// Portable information *should* appear in Config.
type HostConfig struct {
	// Applicable to all platforms
	Binds           []string      // List of volume bindings for this container
	ContainerIDFile string        // File (path) where the containerId is written
	LogConfig       LogConfig     // Configuration of the logs for this container
	NetworkMode     NetworkMode   // Network mode to use for the container
	PortBindings    nat.PortMap   // Port mapping between the exposed port (container) and the host
	RestartPolicy   RestartPolicy // Restart policy to be used for the container
	AutoRemove      bool          // Automatically remove container when it exits
	VolumeDriver    string        // Name of the volume driver used to mount volumes
	VolumesFrom     []string      // List of volumes to take from other container

	// Applicable to UNIX platforms
	CapAdd          strslice.StrSlice // List of kernel capabilities to add to the container
	CapDrop         strslice.StrSlice // List of kernel capabilities to remove from the container
	CgroupnsMode    CgroupnsMode      // Cgroup namespace mode to use for the container
	DNS             []string          `json:"Dns"`        // List of DNS server to lookup
	DNSOptions      []string          `json:"DnsOptions"` // List of DNSOption to look for
	DNSSearch       []string          `json:"DnsSearch"`  // List of DNSSearch to look for
	ExtraHosts      []string          // List of extra hosts
	GroupAdd        []string          // List of additional groups that the container process will run as
	IpcMode         IpcMode           // IPC namespace to use for the container
	Cgroup          CgroupSpec        // Cgroup to use for the container
	Links           []string          // List of links (in the name:alias form)
	OomScoreAdj     int               // Container preference for OOM-killing
	PidMode         PidMode           // PID namespace to use for the container
	Privileged      bool              // Is the container in privileged mode
	PublishAllPorts bool              // Should docker publish all exposed port for the container
	ReadonlyRootfs  bool              // Is the container root filesystem in read-only
	SecurityOpt     []string          // List of string values to customize labels for MLS systems, such as SELinux.
	StorageOpt      map[string]string `json:",omitempty"` // Storage driver options per container.
	Tmpfs           map[string]string `json:",omitempty"` // List of tmpfs (mounts) used for the container
	UTSMode         UTSMode           // UTS namespace to use for the container
	UsernsMode      UsernsMode        // The user namespace to use for the container
	ShmSize         int64             // Total shm memory usage
	Sysctls         map[string]string `json:",omitempty"` // List of Namespaced sysctls used for the container
	Runtime         string            `json:",omitempty"` // Runtime to use with this container

	// Applicable to Windows
	ConsoleSize [2]uint   // Initial console size (height,width)
	Isolation   Isolation // Isolation technology of the container (e.g. default, hyperv)

	// Contains container's resources (cgroups, ulimits)
	Resources

	// Mounts specs used by the container
	Mounts []mount.Mount `json:",omitempty"`

	// MaskedPaths is the list of paths to be masked inside the container (this overrides the default set of paths)
	MaskedPaths []string

	// ReadonlyPaths is the list of paths to be set as read-only inside the container (this overrides the default set of paths)
	ReadonlyPaths []string

	// Run a custom init inside the container, if null, use the daemon's configured settings
	Init *bool `json:",omitempty"`
}
```

```golang
// vendor/github.com/docker/docker/api/types/network/network.go:102

// NetworkingConfig represents the container's networking configuration for each of its interfaces
// Carries the networking configs specified in the `docker run` and `docker network connect` commands
type NetworkingConfig struct {
	EndpointsConfig map[string]*EndpointSettings // Endpoint configs for each connecting network
}

// vendor/github.com/docker/docker/api/types/network/network.go:48

// EndpointSettings stores the network endpoint details
type EndpointSettings struct {
	// Configurations
	IPAMConfig *EndpointIPAMConfig
	Links      []string
	Aliases    []string
	// Operational data
	NetworkID           string
	EndpointID          string
	Gateway             string
	IPAddress           string
	IPPrefixLen         int
	IPv6Gateway         string
	GlobalIPv6Address   string
	GlobalIPv6PrefixLen int
	MacAddress          string
	DriverOpts          map[string]string
}
```

```golang
// cli/command/container/create.go:77

func runCreate(dockerCli command.Cli, flags *pflag.FlagSet, options *createOptions, copts *containerOptions) error {
    // ...
	copts.env = *opts.NewListOptsRef(&newEnv, nil)
	containerConfig, err := parse(flags, copts, dockerCli.ServerInfo().OSType)
	if err != nil {
		reportError(dockerCli.Err(), "create", err.Error(), true)
		return cli.StatusError{StatusCode: 125}
	}
	if err = validateAPIVersion(containerConfig, dockerCli.Client().ClientVersion()); err != nil {
		reportError(dockerCli.Err(), "create", err.Error(), true)
		return cli.StatusError{StatusCode: 125}
	}
	response, err := createContainer(context.Background(), dockerCli, containerConfig, options)
	if err != nil {
		return err
	}
	fmt.Fprintln(dockerCli.Out(), response.ID)
	return nil
}
```
</details>

成功解析参数完成之后经过API版本校验就执行createContainer操作
```golang
// cli/command/container/create.go:192

func createContainer(ctx context.Context, dockerCli command.Cli, containerConfig *containerConfig, opts *createOptions) (*container.ContainerCreateCreatedBody, error) {
	config := containerConfig.Config
	hostConfig := containerConfig.HostConfig
	networkingConfig := containerConfig.NetworkingConfig
	stderr := dockerCli.Err()

	warnOnOomKillDisable(*hostConfig, stderr)
	warnOnLocalhostDNS(*hostConfig, stderr)
```
首先重新从containerConfig中拆出了Config、HostConfig和NetworkingConfig，因为调用dockerd API的时候需要分离的这三个参数。
两个warn分别用于提醒禁用OOM Killer和本地地址DNS的配置，代码如下：
```golang
// cli/command/container/create.go:285

func warnOnOomKillDisable(hostConfig container.HostConfig, stderr io.Writer) {
	if hostConfig.OomKillDisable != nil && *hostConfig.OomKillDisable && hostConfig.Memory == 0 {
		fmt.Fprintln(stderr, "WARNING: Disabling the OOM killer on containers without setting a '-m/--memory' limit may be dangerous.")
	}
}

// check the DNS settings passed via --dns against localhost regexp to warn if
// they are trying to set a DNS to a localhost address
func warnOnLocalhostDNS(hostConfig container.HostConfig, stderr io.Writer) {
	for _, dnsIP := range hostConfig.DNS {
		if isLocalhost(dnsIP) {
			fmt.Fprintf(stderr, "WARNING: Localhost DNS setting (--dns=%s) may fail in containers.\n", dnsIP)
			return
		}
	}
}
```
警告的原因分别是：
- OOM Killer禁用并且未设置memory上限的情况下，容器可能占用过多内存影响主机OS性能甚至崩溃
- 本地地址上的DNS服务器在容器内不会正常工作

继续看createContainer
```golang
// cli/command/container/create.go:192

func createContainer(ctx context.Context, dockerCli command.Cli, containerConfig *containerConfig, opts *createOptions) (*container.ContainerCreateCreatedBody, error) {
	// ...
	containerIDFile, err := newCIDFile(hostConfig.ContainerIDFile)
	if err != nil {
		return nil, err
	}
	defer containerIDFile.Close()
```
这边执行了CIDFile相关的操作，操作本身并不复杂，只是校验hostConfig中的ContainerIDFile是否创建成功了，但是关于这个文件此前并没有关注过。
```golang
	ContainerIDFile string        // File (path) where the containerId is written
```
```golang
// cli/command/container/create.go:175
func newCIDFile(path string) (*cidFile, error) {
	if path == "" {
		return &cidFile{}, nil
	}
	if _, err := os.Stat(path); err == nil {
		return nil, errors.Errorf("Container ID file found, make sure the other container isn't running or delete %s", path)
	}

	f, err := os.Create(path)
	if err != nil {
		return nil, errors.Errorf("Failed to create the container ID file: %s", err)
	}

	return &cidFile{path: path, file: f}, nil
}
```
根据hostConfig中的注释和newCIDFile的逻辑，ContainerIDFile的功能有点类似PIDFile，似乎用于记录容器的ID（在当前只创建了一个文件）

继续看createContainer
```golang
// cli/command/container/create.go:192

func createContainer(ctx context.Context, dockerCli command.Cli, containerConfig *containerConfig, opts *createOptions) (*container.ContainerCreateCreatedBody, error) {
    // ...
    
	var (
		trustedRef reference.Canonical
		namedRef   reference.Named
	)

    // ...

	ref, err := reference.ParseAnyReference(config.Image)
	if err != nil {
		return nil, err
	}
	if named, ok := ref.(reference.Named); ok {
		namedRef = reference.TagNameOnly(named)

		if taggedRef, ok := namedRef.(reference.NamedTagged); ok && !opts.untrusted {
			var err error
			trustedRef, err = image.TrustedReference(ctx, dockerCli, taggedRef, nil)
			if err != nil {
				return nil, err
			}
			config.Image = reference.FamiliarString(trustedRef)
		}
	}
	// ...
```
这段代码的作用是对传入的镜像名称进行解析，得到一个`TrustedReference`，类型是`reference.Canonical`，是由域名、镜像名和摘要组成的足以完全确定镜像的名称。
```golang
// Canonical reference is an object with a fully unique
// name including a name with domain and digest
type Canonical interface {
	Named
	Digest() digest.Digest
}
```

反过来`TrustedReference`可以转换成：
- `normalized name`，是形如`docker.io/library/ubuntu`的名称
- `familiar name`，是形如`ubuntu`的最简名称

还有最后一步准备工作，校验CLI版本是否支持`platform`参数，只有`1.41`版本以上才支持，用于在支持多平台的主机上指定具体平台，包括不同的CPU指令集架构和OS类型
```golang
// cli/command/container/create.go:192

func createContainer(ctx context.Context, dockerCli command.Cli, containerConfig *containerConfig, opts *createOptions) (*container.ContainerCreateCreatedBody, error) {
	// ...
	var platform *specs.Platform
	// Engine API version 1.41 first introduced the option to specify platform on
	// create. It will produce an error if you try to set a platform on older API
	// versions, so check the API version here to maintain backwards
	// compatibility for CLI users.
	if opts.platform != "" && versions.GreaterThanOrEqualTo(dockerCli.Client().ClientVersion(), "1.41") {
		p, err := platforms.Parse(opts.platform)
		if err != nil {
			return nil, errors.Wrap(err, "error parsing specified platform")
		}
		platform = &p
	}
	//...
```

下面就是需要向dockerd发请求的部分了，首先是对设置了`PullImageAlways`的容器配置做处理，会直接先拖取镜像。

```golang
// cli/command/container/create.go:192

func createContainer(ctx context.Context, dockerCli command.Cli, containerConfig *containerConfig, opts *createOptions) (*container.ContainerCreateCreatedBody, error) {
	// ...
	pullAndTagImage := func() error {
		if err := pullImage(ctx, dockerCli, config.Image, opts.platform, stderr); err != nil {
			return err
		}
		if taggedRef, ok := namedRef.(reference.NamedTagged); ok && trustedRef != nil {
			return image.TagTrusted(ctx, dockerCli, trustedRef, taggedRef)
		}
		return nil
	}
	// ...
	if opts.pull == PullImageAlways {
		if err := pullAndTagImage(); err != nil {
			return nil, err
		}
	}
```

pullAndTagImage封装了拖取镜像的逻辑，会调用`pullImage`函数
<details>
<summary>pullImage函数分析</summary>
```golang
// cli/command/container/create.go:105

func pullImage(ctx context.Context, dockerCli command.Cli, image string, platform string, out io.Writer) error {
	ref, err := reference.ParseNormalizedNamed(image)
	if err != nil {
		return err
	}

	// Resolve the Repository name from fqn to RepositoryInfo
	repoInfo, err := registry.ParseRepositoryInfo(ref)
	if err != nil {
		return err
	}

	authConfig := command.ResolveAuthConfig(ctx, dockerCli, repoInfo.Index)
	encodedAuth, err := command.EncodeAuthToBase64(authConfig)
	if err != nil {
		return err
	}

	options := types.ImageCreateOptions{
		RegistryAuth: encodedAuth,
		Platform:     platform,
	}

	responseBody, err := dockerCli.Client().ImageCreate(ctx, image, options)
	if err != nil {
		return err
	}
	defer responseBody.Close()

	return jsonmessage.DisplayJSONMessagesStream(
		responseBody,
		out,
		dockerCli.Out().FD(),
		dockerCli.Out().IsTerminal(),
		nil)
}
```

检查了镜像仓库之后调用Client的ImageCreate向daemon发送请求，通过vendor的方式使用了docker/docker项目中的client

```golang
// vendor/github.com/docker/docker/client/image_create.go:13

// ImageCreate creates a new image based on the parent options.
// It returns the JSON content in the response body.
func (cli *Client) ImageCreate(ctx context.Context, parentReference string, options types.ImageCreateOptions) (io.ReadCloser, error) {
	ref, err := reference.ParseNormalizedNamed(parentReference)
	if err != nil {
		return nil, err
	}

	query := url.Values{}
	query.Set("fromImage", reference.FamiliarName(ref))
	query.Set("tag", getAPITagFromNamedRef(ref))
	if options.Platform != "" {
		query.Set("platform", strings.ToLower(options.Platform))
	}
	resp, err := cli.tryImageCreate(ctx, query, options.RegistryAuth)
	if err != nil {
		return nil, err
	}
	return resp.body, nil
}

func (cli *Client) tryImageCreate(ctx context.Context, query url.Values, registryAuth string) (serverResponse, error) {
	headers := map[string][]string{"X-Registry-Auth": {registryAuth}}
	return cli.post(ctx, "/images/create", query, nil, headers)
}

```
</details>

回到`createContainer`函数的逻辑中，会直接调用一次Client的`ContainerCreate`，如果服务端返回镜像不存在并且容器配置了拖取策略是`PullImageMissing`，会进行一次拖取镜像的逻辑。
```golang
// cli/command/container/create.go:192

func createContainer(ctx context.Context, dockerCli command.Cli, containerConfig *containerConfig, opts *createOptions) (*container.ContainerCreateCreatedBody, error) {
	// ...
	response, err := dockerCli.Client().ContainerCreate(ctx, config, hostConfig, networkingConfig, platform, opts.name)
	if err != nil {
		// Pull image if it does not exist locally and we have the PullImageMissing option. Default behavior.
		if apiclient.IsErrNotFound(err) && namedRef != nil && opts.pull == PullImageMissing {
			// we don't want to write to stdout anything apart from container.ID
			fmt.Fprintf(stderr, "Unable to find image '%s' locally\n", reference.FamiliarString(namedRef))
			if err := pullAndTagImage(); err != nil {
				return nil, err
			}

			var retryErr error
			response, retryErr = dockerCli.Client().ContainerCreate(ctx, config, hostConfig, networkingConfig, platform, opts.name)
			if retryErr != nil {
				return nil, retryErr
			}
		} else {
			return nil, err
		}
	}

	for _, warning := range response.Warnings {
		fmt.Fprintf(stderr, "WARNING: %s\n", warning)
	}
	err = containerIDFile.Write(response.ID)
	return &response, err
```

Client的`ContainerCreate`，通过vendor的方式使用了docker/docker项目中的client
```golang
// vendor/github.com/docker/docker/client/container_create.go:62

// ContainerCreate creates a new container based on the given configuration.
// It can be associated with a name, but it's not mandatory.
func (cli *Client) ContainerCreate(ctx context.Context, config *container.Config, hostConfig *container.HostConfig, networkingConfig *network.NetworkingConfig, platform *specs.Platform, containerName string) (container.ContainerCreateCreatedBody, error) {
	var response container.ContainerCreateCreatedBody

	if err := cli.NewVersionError("1.25", "stop timeout"); config != nil && config.StopTimeout != nil && err != nil {
		return response, err
	}

	// When using API 1.24 and under, the client is responsible for removing the container
	if hostConfig != nil && versions.LessThan(cli.ClientVersion(), "1.25") {
		hostConfig.AutoRemove = false
	}

	if err := cli.NewVersionError("1.41", "specify container image platform"); platform != nil && err != nil {
		return response, err
	}

	query := url.Values{}
	if p := formatPlatform(platform); p != "" {
		query.Set("platform", p)
	}

	if containerName != "" {
		query.Set("name", containerName)
	}

	body := configWrapper{
		Config:           config,
		HostConfig:       hostConfig,
		NetworkingConfig: networkingConfig,
	}

	serverResp, err := cli.post(ctx, "/containers/create", query, body, nil)
	defer ensureReaderClosed(serverResp)
	if err != nil {
		return response, err
	}

	err = json.NewDecoder(serverResp.body).Decode(&response)
	return response, err
}
```
## 2. Docker daemon（docker/docker）
Server注册的路由，注册了`"/containers/create"`，使用`postContainersCreate`进行处理。
```golang
// api/server/router/container/container.go:72

// initRoutes initializes the routes in container router
func (r *containerRouter) initRoutes() {
	r.routes = []router.Route{
		// HEAD
		router.NewHeadRoute("/containers/{name:.*}/archive", r.headContainersArchive),
		// GET
		router.NewGetRoute("/containers/json", r.getContainersJSON),
		router.NewGetRoute("/containers/{name:.*}/export", r.getContainersExport),
		router.NewGetRoute("/containers/{name:.*}/changes", r.getContainersChanges),
		router.NewGetRoute("/containers/{name:.*}/json", r.getContainersByName),
		router.NewGetRoute("/containers/{name:.*}/top", r.getContainersTop),
		router.NewGetRoute("/containers/{name:.*}/logs", r.getContainersLogs),
		router.NewGetRoute("/containers/{name:.*}/stats", r.getContainersStats),
		router.NewGetRoute("/containers/{name:.*}/attach/ws", r.wsContainersAttach),
		router.NewGetRoute("/exec/{id:.*}/json", r.getExecByID),
		router.NewGetRoute("/containers/{name:.*}/archive", r.getContainersArchive),
		// POST
		router.NewPostRoute("/containers/create", r.postContainersCreate),
		router.NewPostRoute("/containers/{name:.*}/kill", r.postContainersKill),
		router.NewPostRoute("/containers/{name:.*}/pause", r.postContainersPause),
		router.NewPostRoute("/containers/{name:.*}/unpause", r.postContainersUnpause),
		router.NewPostRoute("/containers/{name:.*}/restart", r.postContainersRestart),
		router.NewPostRoute("/containers/{name:.*}/start", r.postContainersStart),
		router.NewPostRoute("/containers/{name:.*}/stop", r.postContainersStop),
		router.NewPostRoute("/containers/{name:.*}/wait", r.postContainersWait),
		router.NewPostRoute("/containers/{name:.*}/resize", r.postContainersResize),
		router.NewPostRoute("/containers/{name:.*}/attach", r.postContainersAttach),
		router.NewPostRoute("/containers/{name:.*}/copy", r.postContainersCopy), // Deprecated since 1.8, Errors out since 1.12
		router.NewPostRoute("/containers/{name:.*}/exec", r.postContainerExecCreate),
		router.NewPostRoute("/exec/{name:.*}/start", r.postContainerExecStart),
		router.NewPostRoute("/exec/{name:.*}/resize", r.postContainerExecResize),
		router.NewPostRoute("/containers/{name:.*}/rename", r.postContainerRename),
		router.NewPostRoute("/containers/{name:.*}/update", r.postContainerUpdate),
		router.NewPostRoute("/containers/prune", r.postContainersPrune),
		router.NewPostRoute("/commit", r.postCommit),
		// PUT
		router.NewPutRoute("/containers/{name:.*}/archive", r.putContainersArchive),
		// DELETE
		router.NewDeleteRoute("/containers/{name:.*}", r.deleteContainers),
	}
}
```

`postContainersCreate`的实现

```golang
// api/server/router/container/container_routes.go:460

func (s *containerRouter) postContainersCreate(ctx context.Context, w http.ResponseWriter, r *http.Request, vars map[string]string) error {
	if err := httputils.ParseForm(r); err != nil {
		return err
	}
	if err := httputils.CheckForJSON(r); err != nil {
		return err
	}

	name := r.Form.Get("name")

	config, hostConfig, networkingConfig, err := s.decoder.DecodeConfig(r.Body)
	if err != nil {
		return err
	}
	version := httputils.VersionFromContext(ctx)
	adjustCPUShares := versions.LessThan(version, "1.19")

	// When using API 1.24 and under, the client is responsible for removing the container
	if hostConfig != nil && versions.LessThan(version, "1.25") {
		hostConfig.AutoRemove = false
	}

	if hostConfig != nil && versions.LessThan(version, "1.40") {
		// Ignore BindOptions.NonRecursive because it was added in API 1.40.
		for _, m := range hostConfig.Mounts {
			if bo := m.BindOptions; bo != nil {
				bo.NonRecursive = false
			}
		}
		// Ignore KernelMemoryTCP because it was added in API 1.40.
		hostConfig.KernelMemoryTCP = 0

		// Older clients (API < 1.40) expects the default to be shareable, make them happy
		if hostConfig.IpcMode.IsEmpty() {
			hostConfig.IpcMode = container.IPCModeShareable
		}
	}
	if hostConfig != nil && versions.LessThan(version, "1.41") && !s.cgroup2 {
		// Older clients expect the default to be "host" on cgroup v1 hosts
		if hostConfig.CgroupnsMode.IsEmpty() {
			hostConfig.CgroupnsMode = container.CgroupnsModeHost
		}
	}

	var platform *specs.Platform
	if versions.GreaterThanOrEqualTo(version, "1.41") {
		if v := r.Form.Get("platform"); v != "" {
			p, err := platforms.Parse(v)
			if err != nil {
				return errdefs.InvalidParameter(err)
			}
			platform = &p
		}
	}

	if hostConfig != nil && hostConfig.PidsLimit != nil && *hostConfig.PidsLimit <= 0 {
		// Don't set a limit if either no limit was specified, or "unlimited" was
		// explicitly set.
		// Both `0` and `-1` are accepted as "unlimited", and historically any
		// negative value was accepted, so treat those as "unlimited" as well.
		hostConfig.PidsLimit = nil
	}

	ccr, err := s.backend.ContainerCreate(types.ContainerCreateConfig{
		Name:             name,
		Config:           config,
		HostConfig:       hostConfig,
		NetworkingConfig: networkingConfig,
		AdjustCPUShares:  adjustCPUShares,
		Platform:         platform,
	})
	if err != nil {
		return err
	}

	return httputils.WriteJSON(w, http.StatusCreated, ccr)
}
```

其中主要对版本不支持的参数进行了处理，之后调用`backend.ContainerCreate`进行实际的容器创建操作

<details>
<summary>backend的定义</summary>
```golang
// api/server/router/container/backend.go:1
package container // import "github.com/docker/docker/api/server/router/container"
import (
	"context"
	"io"
	"github.com/docker/docker/api/types"
	"github.com/docker/docker/api/types/backend"
	"github.com/docker/docker/api/types/container"
	"github.com/docker/docker/api/types/filters"
	containerpkg "github.com/docker/docker/container"
	"github.com/docker/docker/pkg/archive"
)

// execBackend includes functions to implement to provide exec functionality.
type execBackend interface {
	ContainerExecCreate(name string, config *types.ExecConfig) (string, error)
	ContainerExecInspect(id string) (*backend.ExecInspect, error)
	ContainerExecResize(name string, height, width int) error
	ContainerExecStart(ctx context.Context, name string, stdin io.Reader, stdout io.Writer, stderr io.Writer) error
	ExecExists(name string) (bool, error)
}

// copyBackend includes functions to implement to provide container copy functionality.
type copyBackend interface {
	ContainerArchivePath(name string, path string) (content io.ReadCloser, stat *types.ContainerPathStat, err error)
	ContainerCopy(name string, res string) (io.ReadCloser, error)
	ContainerExport(name string, out io.Writer) error
	ContainerExtractToDir(name, path string, copyUIDGID, noOverwriteDirNonDir bool, content io.Reader) error
	ContainerStatPath(name string, path string) (stat *types.ContainerPathStat, err error)
}

// stateBackend includes functions to implement to provide container state lifecycle functionality.
type stateBackend interface {
	ContainerCreate(config types.ContainerCreateConfig) (container.ContainerCreateCreatedBody, error)
	ContainerKill(name string, sig uint64) error
	ContainerPause(name string) error
	ContainerRename(oldName, newName string) error
	ContainerResize(name string, height, width int) error
	ContainerRestart(name string, seconds *int) error
	ContainerRm(name string, config *types.ContainerRmConfig) error
	ContainerStart(name string, hostConfig *container.HostConfig, checkpoint string, checkpointDir string) error
	ContainerStop(name string, seconds *int) error
	ContainerUnpause(name string) error
	ContainerUpdate(name string, hostConfig *container.HostConfig) (container.ContainerUpdateOKBody, error)
	ContainerWait(ctx context.Context, name string, condition containerpkg.WaitCondition) (<-chan containerpkg.StateStatus, error)
}

// monitorBackend includes functions to implement to provide containers monitoring functionality.
type monitorBackend interface {
	ContainerChanges(name string) ([]archive.Change, error)
	ContainerInspect(name string, size bool, version string) (interface{}, error)
	ContainerLogs(ctx context.Context, name string, config *types.ContainerLogsOptions) (msgs <-chan *backend.LogMessage, tty bool, err error)
	ContainerStats(ctx context.Context, name string, config *backend.ContainerStatsConfig) error
	ContainerTop(name string, psArgs string) (*container.ContainerTopOKBody, error)

	Containers(config *types.ContainerListOptions) ([]*types.Container, error)
}

// attachBackend includes function to implement to provide container attaching functionality.
type attachBackend interface {
	ContainerAttach(name string, c *backend.ContainerAttachConfig) error
}

// systemBackend includes functions to implement to provide system wide containers functionality
type systemBackend interface {
	ContainersPrune(ctx context.Context, pruneFilters filters.Args) (*types.ContainersPruneReport, error)
}

type commitBackend interface {
	CreateImageFromContainer(name string, config *backend.CreateImageConfig) (imageID string, err error)
}

// Backend is all the methods that need to be implemented to provide container specific functionality.
type Backend interface {
	commitBackend
	execBackend
	copyBackend
	stateBackend
	monitorBackend
	attachBackend
	systemBackend
}
```
</details>
daemon就是实现了Backend接口的一个类型，`ContainerCreate`公开方法会调用`containerCreate`私有方法。
```golang
// daemon/create.go:41

// ContainerCreate creates a regular container
func (daemon *Daemon) ContainerCreate(params types.ContainerCreateConfig) (containertypes.ContainerCreateCreatedBody, error) {
	return daemon.containerCreate(createOpts{
		params:                  params,
		managed:                 false,
		ignoreImagesArgsEscaped: false})
}
```
下面是containerCreate的具体实现
```golang
func (daemon *Daemon) containerCreate(opts createOpts) (containertypes.ContainerCreateCreatedBody, error) {
	start := time.Now()
	if opts.params.Config == nil {
		return containertypes.ContainerCreateCreatedBody{}, errdefs.InvalidParameter(errors.New("Config cannot be empty in order to create a container"))
	}

	warnings, err := daemon.verifyContainerSettings(opts.params.HostConfig, opts.params.Config, false)
	if err != nil {
		return containertypes.ContainerCreateCreatedBody{Warnings: warnings}, errdefs.InvalidParameter(err)
	}

	if opts.params.Platform == nil && opts.params.Config.Image != "" {
		if img, _ := daemon.imageService.GetImage(opts.params.Config.Image, opts.params.Platform); img != nil {
			p := platforms.DefaultSpec()
			imgPlat := v1.Platform{
				OS:           img.OS,
				Architecture: img.Architecture,
				Variant:      img.Variant,
			}

			if !images.OnlyPlatformWithFallback(p).Match(imgPlat) {
				warnings = append(warnings, fmt.Sprintf("The requested image's platform (%s) does not match the detected host platform (%s) and no specific platform was requested", platforms.Format(imgPlat), platforms.Format(p)))
			}
		}
	}

	err = verifyNetworkingConfig(opts.params.NetworkingConfig)
	if err != nil {
		return containertypes.ContainerCreateCreatedBody{Warnings: warnings}, errdefs.InvalidParameter(err)
	}

	if opts.params.HostConfig == nil {
		opts.params.HostConfig = &containertypes.HostConfig{}
	}
	err = daemon.adaptContainerSettings(opts.params.HostConfig, opts.params.AdjustCPUShares)
	if err != nil {
		return containertypes.ContainerCreateCreatedBody{Warnings: warnings}, errdefs.InvalidParameter(err)
	}

	ctr, err := daemon.create(opts)
	if err != nil {
		return containertypes.ContainerCreateCreatedBody{Warnings: warnings}, err
	}
	containerActions.WithValues("create").UpdateSince(start)

	if warnings == nil {
		warnings = make([]string, 0) // Create an empty slice to avoid https://github.com/moby/moby/issues/38222
	}

	return containertypes.ContainerCreateCreatedBody{ID: ctr.ID, Warnings: warnings}, nil
}
```