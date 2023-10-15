# everythink-about-dot-net

## Entity Framework nedir?
Öncelikle Entity Framework nedir buna değinmek istiyorum. Entity Framework, Microsoft tarafından geliştirilen ORM aracıdır. Temel olarak uygulama ve veritabanı arasında köprü görevi gören bir araçtır.

## ORM nedir?
Object Relational Mapping(ORM), nesne dayalı programlama paradigması kullanarak bir veritabanındaki verileri sorgulamamıza ve değiştirmemize olanak tanıyan bir tekniktir. Veritabanındaki tabloları, sınıflara çevirir bu sayede kod yazarken veritabanında daha az zaman harcarız. Daha az SQL sorgusu yazmamıza olanak sağlar. Buna bağlı olarak uygulama geliştirme maliyetini düşürür. 

## Code First nedir?
Code First yaklaşımı basit olarak sınıflar ile veritabanımızı oluşturmamızı sağlar. Yani modelimizi C # veya VB.Net sınıflarını kullanarak tanımlamanıza olanak tanır. Code First yaklaşımı genelde var olmayan bir veritabanını hedeflemektedir.

## Fluent API & Data Annotations

### Fluent Api, Entity Framework Code First yaklaşımı ile kullanacağımız veri tabanı sınıflarını(entity) ve ilişkilerini yapılandırabilmemizi sağlayan bir yoldur. Entity dosyalarımızda Data Annotation‘ları kullanarak da gerekli işlemleri sağlayabiliriz. Fakat Fluent Api ile bu ayarlamaları yapmamız daha anlaşılır, daha temiz olacaktır. Fluent Api kullanımını sağlamak için ilgili Class dosyalarımızı tanımlayarak içerisinde de tanımlamalarımızı, özelliklerimizi tanımlayacağız.
Projeyi oluşturduktan sonra içerisine 3 adet klasör eklemesi yapıyoruz.

Entities
Data
Mappings.

```c#
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }
}

public class FluentApiContext : DbContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer(@"ConnectionStringBilginiz");
    }

    public DbSet<Product> Products { get; set; }
}

public class ProductMap : IEntityTypeConfiguration<Product>
{
    public void Configure(EntityTypeBuilder<Product> builder)
    {
        builder.HasKey(I => I.Id);
        
        builder.Property(I => I.Id)
               .UseIdentityColumn();

        builder.Property(I => I.Name)
               .HasMaxLength(100)
               .HasColumnType("varchar")
               .IsRequired();
    }
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
     modelBuilder.ApplyConfiguration(new ProductMap());
}

public class FluentApiContext : DbContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer(@"ConnectionStringBilginiz");
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.ApplyConfiguration(new ProductMap());
    }

    public DbSet<Product> Products { get; set; }
}
```

### Data Annotation, System.ComponentModel.DataAnnotations namespace’i altında bulunan Data Annotation’lar genellikle Entity Framework(Core) ile tercih edilmektedirler ve entity model üzerinde gerekli validasyonel ayarların attributelar aracılığıyla yapılmasını sağlayarak işimizi oldukça kolaylaştırmaktadırlar.

```
class Customer
{
    public int Id { get; set; }
    [Required(ErrorMessage = "Lütfen adınızı boş geçmeyiniz")]
    public string Name { get; set; }
    [Required(ErrorMessage = "Lütfen soyadınızı boş geçmeyiniz.")]
    [StringLength(15, ErrorMessage = "Lütfen soyadınızı 3 ile 15 karakter arasında yazınız.", MinimumLength = 3)]
    public string Surname { get; set; }
    [Required(ErrorMessage = "Lütfen doğum tarihinizi boş geçmeyiniz.")]
    public DateTime BornDate { get; set; }
}

```

### Peki Neden FluentValidation Kullanmalıyız?
Data annotationlar yapısal olarak evet kullanışlı ve pratik olabilirler. Ama işlevsel olarak SOLID prensiplerine aykırı bir yazım stiline sahiptirler. Bunun için yukarıda oluşturduğumuz Customer modelini aşağıya tekrar alıp incelersek eğer;


```
class Customer
{
    public int Id { get; set; }
    [Required(ErrorMessage = "Lütfen adınızı boş geçmeyiniz")]
    public string Name { get; set; }
    [Required(ErrorMessage = "Lütfen soyadınızı boş geçmeyiniz.")]
    [StringLength(15, ErrorMessage = "Lütfen soyadınızı 3 ile 15 karakter arasında yazınız.", MinimumLength = 3)]
    public string Surname { get; set; }
    [Required(ErrorMessage = "Lütfen doğum tarihinizi boş geçmeyiniz.")]
    public DateTime BornDate { get; set; }
}

```

görüldüğü üzere Customer sadece bir entity olarak tasarlanan modeldir. İşlevi sadece bu olması gereken Customer sınıfı içerisinde ayriyetten data annotation sayesinde validasyon ayarlaması yapılmıştır. Buda SOLID prensiplerinden Tek Sorumluluk Prensibi(Single Responsibility Principle – SRP)‘ne aykırıdır! İşte bu durumda prensibe uygun validasyonel yapılanma inşa edebilmek için tüm sorumluluğu tek bir sınıfın üstleneceği bir stratejiyi benimsememiz gerekmektedir ve bunun içinde en kullanışlı tasarıma sahip kütüphane olan FluentValidation’ı tercih etmekteyiz.

Entity Framework(Core) ile tercih edilen Data Annotation validasyonları Tek Sorumluluk Prensibi(Single Responsibility Principle – SRP)‘ne aykırıdır!

### IEnumerable ve IQueryable
 IEnumerable: IEnumerable tipi veriyi önce belleğe atıp ardından bellekteki bu veri üzerinden belirtilen koşulları çalıştırır ve veriye uygular.
 IQueryable: IQueryable tipinde ise belirtilen sorgular direk olarak server üzerinde çalıştırılır ve dönüş sağlar. Ayrıca bu tip IEnumerable tipini implement ettiği için IEnumerable’ın tüm özelliklerini kullanabilir.

 Aslında açıklamalara bakacak olursak ikisi arasındaki farkı çok rahat görebiliriz. IQueryable bize database vb. veri depolarında yapılan sorgulamalar da işlevsellik sağlarken, IEnumerable ise bize bir koleksiyon üzerinde sorgulama yapmak için olanak sağlar.

 Gerçek hayat kullanımına dair bir örnek üzerinden ilerlersek elimizde çok büyük bir kayır olduğunu düşünelim. Örnek veriyorum 1 milyon kaydımız var ve biz burada bir sorgu yapmak istiyoruz. Şart olarak ise PaymentStatus=true verdiğimizi düşünelim.

IEnumerable mantığı üzerinden gidersek 1 milyon veri öncelikle belleğe alır ve ardından sorgumuzu uygular. Biraz daha açacak olursak bellek öncelikle IEnumerable için yer açıyor ve datayı buraya alıyor. Daha sonra şartımız where komutu ile burada sorgulanıyor.

IQueryable ise öncelikle belirtiğimiz koşula göre bir sorgu uygulayıp bunula database’e gidiyor gerekli verileri aldıktan sonra bize dönüş sağlıyor.


Differences:

IEnumerable
* Enumerable exists in the System.Collections namespace.
* IEnumerable is suitable for querying data from in-memory collections like List, Array and so on.
* While querying data from the database, IEnumerable executes "select query" on the server-side, loads data in-memory on the client-side and then filters the data.
* IEnumerable is beneficial for LINQ to Object and LINQ to XML queries.

IQueryable
* IQueryable exists in the System.Linq Namespace.
* IQueryable is suitable for querying data from out-memory (like remote database, service) collections.
* While querying data from a database, IQueryable executes a "select query" on server-side with all filters.
* IQueryable is beneficial for LINQ to SQL queries.

