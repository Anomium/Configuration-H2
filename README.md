# Introduccion al gestor de base de datos H2 para Test Unitario 🚀
### Motivo de Instalacion de una segunda base de datos 📋
_El motivo principal de otra base de datos es para poder guardar, editar y realizar consultas a ella misma a nivel de test unitario. Con esto se evita el problema de estar creando registros falsos en produccion._
### ¿Que es la base de datos H2?

_H2 es un motor de bases de datos SQL que se escribe en Java que implefmenta la API de JDBC. Se incluye una aplicacion de consola basada en navegador._

_Limitaciones en el soporte de la base de datos H2:_

* _Solo para un uso de desarrollo._
* _No esta soportado durante el tiempo de ejecucion._
* _No se pueden compilar archivos EAR para esta base de datos._
* _No se puede ejecutar el destino *configure* mientras se esta utilizando esta base de datos. Este destino configura automaticamente el servidor de aplicaciones._

### Instalacion previa 🔧
_Configuracion para evitar que nuestra consulta sea a la base de datos de produccion (en nuestro caso es GCP)._

```
spring.cloud.gcp.pubsub.enabled = false
spring.cloud.gcp.sql.enabled = false
spring.cloud.gcp.datastore.enabled = false
```
_Configuraciones adicionales de Hibernte para h2_
```
hibernate.dialect=org.hibernate.dialect.H2Dialect
hibernate.show_sql=true
hibernate.hbm2ddl.auto=create-drop
hibernate.cache.use_second_level_cache=true
hibernate.cache.use_query_cache=true
hibernate.cache.region.factory_class=org.hibernate.cache.ehcache.EhCacheRegionFactory
```
### Configuraciones del perfil para test unitario🔧
```
@Configuration
@EnableJpaRepositories(
    basePackages = {"org.example.example.example.infraestructure.repository.port"})
@EntityScan("org.example.example.example.infraestructure.repository.dto.*")
@EnableTransactionManagement
@Profile("test")
@PropertySource("classpath:application-test.properties")
public class H2TestProfileJPAConfig {

  @Autowired
  private Environment env;

  @Bean
  @Profile("test")
  public DataSource dataSource() {
    final DriverManagerDataSource dataSource = new DriverManagerDataSource();
    dataSource.setDriverClassName("org.h2.Driver");
    dataSource.setUrl("jdbc:h2:mem:db;DB_CLOSE_DELAY=-1");
    dataSource.setUsername("sa");
    dataSource.setPassword("sa");

    return dataSource;
  }

  @Bean
  public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
    final LocalContainerEntityManagerFactoryBean em = new LocalContainerEntityManagerFactoryBean();
    em.setDataSource(dataSource());
    em.setPackagesToScan(new String[] {"org.example.example.example.infraestructure.repository.dto"});
    em.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
    em.setJpaProperties(additionalProperties());
    return em;
  }

  @Bean
  JpaTransactionManager transactionManager(final EntityManagerFactory entityManagerFactory) {
    final JpaTransactionManager transactionManager = new JpaTransactionManager();
    transactionManager.setEntityManagerFactory(entityManagerFactory);
    return transactionManager;
  }

  final Properties additionalProperties() {
    final Properties hibernateProperties = new Properties();

    hibernateProperties.setProperty("hibernate.hbm2ddl.auto",
        env.getProperty("hibernate.hbm2ddl.auto"));
    hibernateProperties.setProperty("hibernate.dialect", env.getProperty("hibernate.dialect"));
    hibernateProperties.setProperty("hibernate.show_sql", env.getProperty("hibernate.show_sql"));

    return hibernateProperties;
  }
}
```
_Activacion del perfil de Test unitario_

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {H2TestProfileJPAConfig.class, CoreApplication.class})
@EnableConfigurationProperties
@ActiveProfiles("test")
public class ReporteServiceTest {
```
### Ejecucion de los test unitarios utilizando H2 🔧

_Primeramente creamos el objeto con todos los datos que necesitaremos._

```
listReport = new ArrayList<Reporte>();
    Reporte reporte = new Reporte();
    reporte.setId(28873560L);
    reporte.setNumeroPrescripcion("655262");
    reporte.setTecnologia(new Tecnologia());
    reporte.setPaciente(new Paciente());
    reporte.setEntrega(new Entrega());
```
_Una vez tengamos el objeto creado lo guardamos utilizando el metodo save del repositorio que en nuestro caso extiende de CrudRepository_
* _Se guarda automaticamente en el H2 debido a que habiamos activado el perfil de test._
```
repository.save(reporte);
```
_Luego podremos consultar y hacer las acciones necesarias para continuar nuestro test unitario._
