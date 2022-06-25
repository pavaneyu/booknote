## 用函数式编程而非注解实现WebFlux(使用模拟数据)
1.服务器绑定adapter,adapter绑定路由 
2.路由绑定handler   ,通过Url类型  导航到 handler 具体方法 ,返回一个RouterFunction<ServerResponse>
3.handler 绑定 一个service ,根据Url里面参数,用service获取一个Flux
再把Flux统一转成Mono<ServerResponse>
```

public class Server {

    public static void main(String[] args) throws IOException {
        new Server().createReactorServer();
        System.out.println("enter to exit");
        System.in.read();
    }


    /**
     * 路由 指明由地址栏解析的请求具体 去执行那个方法  bind handler  ,
     * @return
     */
    public RouterFunction<ServerResponse> routerFunction() {
        EmpHandler handler = new EmpHandler(new MyService());

        return RouterFunctions.route(
                RequestPredicates.GET("/emps/id={id}").and(RequestPredicates.accept(MediaType.APPLICATION_JSON)),
                handler::getEmpByID
        ).andRoute(
                RequestPredicates.GET("/emps").and(RequestPredicates.accept(MediaType.APPLICATION_JSON)),
                handler::getAllEmps
        );
    }

    /**
     * 适配器 binds with router
     */
    public void createReactorServer() {

        HttpHandler httpHandler = RouterFunctions.toHttpHandler(routerFunction());
        ReactorHttpHandlerAdapter adapter = new ReactorHttpHandlerAdapter(httpHandler);

        //server binds adapter
        HttpServer.create().handle(adapter).bindNow();
    }

}
```
```

public class EmpHandler {
    private final EmpService empService;  //handler binds with service

    public EmpHandler(EmpService empService) {//这里的形参是接口,传入要传入一个具体实现类
        this.empService = empService;
    }

    public Mono<ServerResponse> getEmpByID(ServerRequest request) {
        int id = Integer.parseInt(request.pathVariable("id"));
        Mono<Employee> employeeMono = empService.getEmpByID(id);
//        employeeMono.flatMap( employee -> ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(employee,employee.getClass()))
        return employeeMono.flatMap( employee -> ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(fromObject(employee)))
                .switchIfEmpty(ServerResponse.notFound().build());
    }

    public Mono<ServerResponse> getAllEmps(ServerRequest request) {
        Flux<Employee> allEmps = empService.getAllEmps();
        return ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(allEmps, Employee.class);
    }

    public Mono<ServerResponse> addEmp(ServerRequest request) {
        Mono<Employee> employeeMono = request.bodyToMono(Employee.class);
        return ServerResponse.ok().build(empService.addEmp(employeeMono));
    }


}
```
@Repository
public class MyService implements EmpService {

    private List<Employee> list = new ArrayList<>();
    
    {
        list.add(new Employee(1, "a", "A"));
        list.add(new Employee(2, "b", "B"));
        //list.add(new Employee(3, "c", "C"));
    }
    
    public List<Employee> getList() {
        return list;
    }


    @Override
    public Mono<Employee> getEmpByID(int id) {
        for (Employee employee : list) {
            if (employee.getId() == id) {
                return Mono.justOrEmpty(employee);
            }
        }
        return null;
    }
    
    @Override
    public Flux<Employee> getAllEmps() {
        return Flux.fromIterable(list);
    }
    
    @Override
    public Mono<Void> addEmp(Mono<Employee> empMono) {
        return empMono.doOnNext(employee -> list.add(employee)).thenEmpty(Mono.empty());
    }
```