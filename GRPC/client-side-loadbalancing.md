### **Here we will discussing client-side loadbalancing**
There is two policy in GRPC
- PickFirst
- round_robin

#### **1. pick first**
 it is by default loadbalancing is there you no need to do any think in grpc
#### **2. Round Robin**
For that we need to do some setup which we will ndo here
- first we need to register NameResolverRegistry

#### Example:
```java
NameResolverRegistry.getDefaultRegistry().register(new ExampleNameResolverProvider());
```
creating ExampleNameResolverProvider.java file

```java
public class ExampleNameResolverProvider extends NameResolverProvider {
    @Override
    public NameResolver newNameResolver(URI targetUri, NameResolver.Args args) {
        return new ExampleNameResolver(targetUri);
    }

    @Override
    protected boolean isAvailable() {
        return true;
    }

    @Override
    protected int priority() {
        return 5;
    }

    @Override
    // gRPC choose the first NameResolverProvider that supports the target URI scheme.
    public String getDefaultScheme() {
        return "example";
    }
}
```
creating ExampleNameResolver.java file
```java
public class ExampleNameResolver extends NameResolver {

    static private final int[] SERVER_PORTS = {9091, 9092};
    private Listener2 listener;

    private final URI uri;

    private final Map<String,List<InetSocketAddress>> addrStore;

    public ExampleNameResolver(URI targetUri) {
        this.uri = targetUri;
        // This is a fake name resolver, so we just hard code the address here.
        addrStore = ImmutableMap.<String,List<InetSocketAddress>>builder()
            .put("lb.example.grpc.io",
                Arrays.stream(SERVER_PORTS)
                    .mapToObj(port->new InetSocketAddress("localhost",port))
                    .collect(Collectors.toList())
            )
                .build();
    }

    @Override
    public String getServiceAuthority() {
        // Be consistent with behavior in grpc-go, authority is saved in Host field of URI.
        if (uri.getHost() != null) {
            return uri.getHost();
        }
        return "no host";
    }

    @Override
    public void shutdown() {
    }

    @Override
    public void start(Listener2 listener) {
        this.listener = listener;
        this.resolve();
    }

    @Override
    public void refresh() {
        this.resolve();
    }

    private void resolve() {
        List<InetSocketAddress> addresses = addrStore.get(uri.getPath().substring(1));
        try {
            List<EquivalentAddressGroup> equivalentAddressGroup = addresses.stream()
                    // convert to socket address
                    .map(this::toSocketAddress)
                    // every socket address is a single EquivalentAddressGroup, so they can be accessed randomly
                    .map(Arrays::asList)
                    .map(this::addrToEquivalentAddressGroup)
                    .collect(Collectors.toList());

            ResolutionResult resolutionResult = ResolutionResult.newBuilder()
                    .setAddresses(equivalentAddressGroup)
                    .build();

            this.listener.onResult(resolutionResult);

        } catch (Exception e){
            // when error occurs, notify listener
            this.listener.onError(Status.UNAVAILABLE.withDescription("Unable to resolve host ").withCause(e));
        }
    }

    private SocketAddress toSocketAddress(InetSocketAddress address) {
        return new InetSocketAddress(address.getHostName(), address.getPort());
    }

    private EquivalentAddressGroup addrToEquivalentAddressGroup(List<SocketAddress> addrList) {
        return new EquivalentAddressGroup(addrList);
    }
}
```
creating connection 
```java
 NameResolverRegistry.getDefaultRegistry().register(new ExampleNameResolverProvider());

        String target = String.format("%s:///%s", exampleScheme, exampleServiceName);

 logger.info("Change to round_robin policy");
        ManagedChannel channel = ManagedChannelBuilder.forTarget(target)
                .defaultLoadBalancingPolicy("round_robin")
                .usePlaintext()
                .build();
```
---
