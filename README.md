# UYUM FRAMEWORK TEKNİK DÖKÜMAN 
## UYUM FRAMEWORK’ÜN GENEL YAPISI NASILDIR?

Uyum Framework’te veritabanı işlemleri objeler üzerinden yapılmaktadır.  Ekran tasarımları XML ile yapılmaktadır. Standart ekranlarda her bir ekrana karşılık gelen bir obje vardır. Bir modüle ait objeler bir dll’in içinde toplanır. Module ait yazılan custom kodlar başka dll de yer alır. XML ve Javascriptler, Senfoni projesi içinde ilgili dizinde yer alır. Bir de komut yapısı vardır. Ekranlar açılırken parametre olarak komut geçilir.  Kullanıcının komuta yetkisi var ise ekran açılmaktadır. Devexpress kontrol’leri kullanılmıştır. 

## OBJECT VE COLLECTION NEDİR?

Objeler ekranın datasource’udurlar. Veritabanından çekilen verileri tutmaya ve aynı zamanda bu verileri, objenin ana tablosuna yazmasına sağlar.  Tablodaki alanları özellikleri (tipi, uzunluğu, açıklaması, index gibi) object’te belirlenir. Collection’da tablo adı ve Join bilgileri yer almaktadır. 

Örnek bir İlçe objesi aşağıda yer almaktadır.
```cs 
using System;
using System.Collections.Generic;
using System.Runtime.InteropServices;
using System.Text;
using Uyum.Net;
using Uyum.Objects;

namespace GNL
{
    [Guid("6d510caa-ce82-4c88-b00c-5400fb2d7e45")]
    public class Town : UyumObjectBase
    {
		private DateTime _CreateTime = DateTime.Now;
		private DateTime _UpdateTime = DateTime.Now;
		private int _CreateUserId = 0;
		private int _UpdateUserId = 0;
		private string _TownName = string.Empty;
		private int _CityId = 0;
    		private string _CityName = string.Empty;
		private int _CountryId = 0;
             private string _CountryName = string.Empty;
             private string _Longitude = string.Empty; 
             private string _Latitude = string.Empty;  

		public Town()
		{
			
		}

		[DataInt32("TOWN_ID", IsIdentity=true)]
		[UyumPrimaryKey(0)]
		public override int Id
		{
			get { return base.Id; }
			set { base.Id = value; }
		}

		[DataTime("CREATE_TIME")]
		public DateTime CreateTime
		{
			get { return _CreateTime; }
			set { _CreateTime = value; }
		}


		[DataTime("UPDATE_TIME")]
		public DateTime UpdateTime
		{
			get { return _UpdateTime; }
			set { _UpdateTime = value; }	
}

		[DataInt32("CREATE_USER_ID")]
		public int CreateUserId
		{
			get { return _CreateUserId; }
			set { _CreateUserId = value; }
		}

		[DataInt32("UPDATE_USER_ID")]
		public int UpdateUserId
		{
			get { return _UpdateUserId; }
			set { _UpdateUserId = value; }
		}
       
       [DataString("TOWN_NAME", Length=40), TitleCaption("İlçe")]
             [UyumIndex(Name = "UN_GNLD_TOWN", IsUnique = true)]
		public string TownName
		{
			get { return _TownName; }
			set { _TownName = value; }
		}

		[DataInt32("CITY_ID")]
             [UyumIndex(Name = "UN_GNLD_TOWN", IsUnique = true)]
		public int CityId
		{
			get { return _CityId; }
			set { _CityId = value; }
		}

        [DataString("CITY_NAME", Flags = ColumnFlags.None, Length = 40, TableAlias = "GNLD_CITY"), TitleCaption("İl")]
        public string CityName
        {
            get { return _CityName; }
            set { _CityName = value; }
        }

		[DataInt32("COUNTRY_ID")]
             [UyumIndex(Name = "UN_GNLD_TOWN", IsUnique = true)]
		public int CountryId
		{
			get { return _CountryId; }
			set { _CountryId = value; }
		}

        [DataString("COUNTRY_NAME", Flags = ColumnFlags.None, TableAlias = "GNLD_COUNTRY"), TitleCaption("Ülke")]
        public string CountryName
        {
            get { return _CountryName; }
            set { _CountryName = value; }
        }
	}
}
```

Tablonun primary key alanını **[UyumPrimaryKey(0)]** attribute ile ve otomatik artan olduğunu belirtmek için **IsIdentity=true** kullanıyoruz. 

Ana tablosundan değil de başka bir tablodan alan geliyor ise **Flags = ColumnFlags.None** attribute ile belirtilir. **TableAlias** ile hangi tablodan geldiği yazılır. **Alias** attribute’ü ile field alias verebiliriz. 
**UyumIndex** attribute ile index tanımlayabiliriz. Bir index’i birden fazla alanda kullanacak iseniz, onlarada aynı index name’i veriniz. Index Unique olacak ise IsUnique = true kullanınız.
İlçe objesini Collection’ı aşağıdaki gibi tanımlanmıştır.
```cs 
using System;
using System.Collections.Generic;
using System.Runtime.InteropServices;
using System.Text;
using Uyum.Objects;

namespace GNL
{
	[Guid("fdece7ef-9ba8-45a1-9bab-b5db5ac7423e")]
	[UyumTable("GNLD_TOWN", Database="Uyumsoft", InsertStoredProcedure="GNL_TOWN_I", UpdateStoredProcedure="GNL_TOWN_U", DeleteStoredProcedure="GNL_TOWN_D", SelectStoredProcedure="GNL_TOWN_S")]
    [UyumJoin(JoinType.LeftOuterJoin, "GNLD_TOWN", "COUNTRY_ID", "GNLD_COUNTRY", "COUNTRY_ID", Order = 0, DeleteAction = ChangeAction.ThrowException)]
    [UyumJoin(JoinType.LeftOuterJoin, "GNLD_TOWN", "CITY_ID", "GNLD_CITY", "CITY_ID", Order = 1, DeleteAction = ChangeAction.ThrowException)]
	public class TownCollection : UyumObjectCollection
	{
		public TownCollection()
		{
			base._ItemType = typeof(Town);
		}

		public override IUyumObject CreateItem()
		{
			return new Town();
		}

		public Town this[int index]
		{
			get { return base.innerList[index] as Town; }
		}
	}
}
```
**UyumTable** attribute ile tablo adı belirtilir. Stored procedure isimleri belirtilir. 
**UyumJoin** attribute ile ekrana veri gelecek ana tablo ile ilişkili joinler belirtilir. İçeriği Join tipi, tablo adı ve field adıdır.
**DeleteAction** attribute joinde yazılır. *Foreignkey* kavramının object üzerinde yapmamızı sağlar. Bu örnekte **GNLD_CITY** (İl tablosu) silindiği zaman ilçe tablosunda kullanılıp kullanılmadığına bakara. Eğer kullandı ise hata fırlatır.


Object içerisinde detay kaydı var ise detay kaydı aşağıdaki gibi tanımlanır.
```cs
[Guid("b76855cf-c9b3-4f38-b62f-5cd3bc3248e4")]
public class FinM : UyumObjectBase
{
        private int _CreateUserId = 0;
        private int _UpdateUserId = 0;
        ……………………………..
        ……………………………..
FinDCollection _FinDCollection = new FinDCollection();    
             [UyumDetailObject("Id", "FinMId")]
      [Server]
      public FinDCollection FinDCollection
      {
           get
           {
               return _FinDCollection;
           }
           set
           {
               _FinDCollection = value;
           }

       }
}
```
**UyumDetailObject** attribute’ün açıklaması şöyledir.

*MasterKey* : Master’ın detay objesine bağlı olan property name’i.
*DetailKey* : Detayı’ın master objesine bağlı olan property name’i.
*MasterProperty* : Detayın detayında geçerlidir. Hangi detaya bağlı olduğunu gösteren propertyname’dir.
*SaveIgnore* : Master kaydedilirken detayın altyapı tarafından kaydedilmesi engellenir.
*DeleteIgnore* : Master silinirken detayı’ın altyapı tarafından otomatik silinmesi engellenir.
*IsAllLoad* : Detayın detayında geçerlidir. Master yüklenirken tüm detay detay kayıtlarınında yüklenmesini sağlar. Aksi halde sadece detayın ilk kaydına ait olan detaylar yüklenir.

Objenin içinde Insert, Update ve Delete metodları yazabilirsiniz. Objenin Insert, Update ve Delete methodlarını çağırdığınızda bu methodların içine düşer. Bu methodlarda yapmak istediğiniz diğer işlemleri yapabilirsiniz. Örneğin fatura kaydedilirken, fatura objesinin içerisinde finansa kayıt atabilirsiniz.
Detay objelerinin değerlerini bu metodlar içerisinde okurken **Modified** alana dikkat ediniz. **Modified** alanı sıfır ve Id sıfır veya sıfrdan küçükse yeni kayıt, Modified alanı bir ise düzeltilmiş , iki ise bu kayıt ekrandan silinmiş olduğunu anlayabilirsiniz…


##	XML VE İÇERİSİNDE KULLANILAN ATTRIBUTE’LERİN ANLAMI NEDİR?

XML kolayca ekran tasarlamak için geliştirilen bir yapıdır. 
Aşağıda örnek bir ilçe ekranının ekranı mevcuttur.
```xml
<root MainCode="GNL10351" Caption="İlçe Tanımı" MainObject="GNL.TownCollection,GNL ">
  <tabcontrol Visibility="True">
    <tabpage Caption="İlçe Tanımı">
      <section Caption="İlçe Tanımı" CaptionVisibility="True" Visibility="True">
        <row>
          <cell colspan="6">
            <control FieldName="TownName" ControlType="TextEdit" MaxLength="40" Caption="İlçe">
            </control>
          </cell>
        </row>
        <row>
          <cell colspan="6">
            <control FieldName="CityName" ControlType="ButtonEdit" Caption="İl" ControlRequired="true" ControlEnabled="true" ControlVisible="True" ControlSingleLine="true">
              <DataSource SourceType="Command" Source="CityCollection.Show" Filter="" FilterValues="" RelatedProperty="" ReturnProperties="Id;CityName;CountryId;CountryName" ReturnedProperties="CityId;CityName;CountryId;CountryName" OrderByProperty="" ListPropertyName="CityName" ProcessTypeMode="1" />
            </control>
          </cell>
        </row>
        <row>
          <cell colspan="6">
            <control FieldName="CountryName" ControlType="TextEdit" Caption="Ülke" ControlRequired="true" ControlEnabled="false" ControlVisible="True" ControlSingleLine="true">
            </control>
          </cell>
        </row>
        <row>
          <cell colspan="6">
            <control FieldName="Latitude" ControlType="TextEdit" Caption="Enlem" MaxLength="15" ControlRequired="False" ControlEnabled="true" ControlVisible="True" ControlSingleLine="true" ServerAttribute="TabIndex=8">
            </control>
          </cell>
        </row>
      </section>
    </tabpage>
  </tabcontrol>
  <hidden>
    <control FieldName="CountryId" ControlType="HiddenEdit" Caption="Ülke ID" ControlRequired="True" ControlEnabled="true" ControlVisible="True" ControlSingleLine="true">
    </control>
    <control FieldName="CityId" ControlType="HiddenEdit" Caption="İl ID" ControlRequired="false" ControlEnabled="true" ControlVisible="True" ControlSingleLine="true">
    </control>
  </hidden>
</root>
```

Ekran tasarlanırken, öncelikle ekranda kaç *tabcontrol* kullanacak iseniz o kadar *tabcontrol* konur. <u>En az bir tane olmak zorundadır</u>. Tabcontrolun içerisine <u>en az bir tane section</u> olmak zorundadır. *Section’ı* tablo olarak düşünebilirsiniz. Kontrolleri bu tablonun içerisine koyacağız. Öncelikle bu tabloyu kaç kolona bölecek iseniz ColumnCount attibute ile bunu yazınız. Yazmazsanız varsayılan olarak 6’ya böler. row attribute ile section’ı satırlara ayırıyoruz. cell attribute ile bu satırları hücrelere ayırıyouz. En sonda da bu hücrelerin içine control lerimizi koyuyoruz.
Şimdi ayrıntılı olarak her bir attribute’u inceleyelim…
1.	root : Ekran ile ilgili genel bilgiler tanımlanır. 
  *	MainCode : Ekranın pagecodu yazılır. Zorunludur.
  *	Caption : Ekranın başlığı tanımlanır. Zorunludur.
  *	MainObject : Ekranın objesi tanımlanır. Object ile bind işlemi otomatik yapılacak ise zorunludur.
  *	CustomAttribute : Body kısmına attribute eklemek için kullanılır.
  *	InVisibleCommandButton : Ekranın üstündeki kaydet, kaydet yeni gibi butonları gizler. Gizlemek istenen butonlar noktalı virgül ile ayrılarak yazılır. Değerler  Save, SaveClose, SaveAndInsert, Cancel, IsUpdate, IsCopy dir.
  *	FillList : İlk yüklemede gridin boş gelip gelmeyeceğini karar verir.
  *	UnLoad : Sayfa kapanırken çalışacak javascript fonksiyon belirtilir.
  *	CallbackShowDialog : Sayfada bir callback olursa otomatik diyalog çıkmasını sağlar.

2.	script : Javascript fonksiyonu veya dosyası yüklemeye yarar.
  *	ScriptCode : Javascript metodu direk olarak ekrana yazılabilir. Fakat bu tavsiye edilmez. Bunun yerine bir dosyaya yazıp bu dosya adının verilmesi gerekir.
  *	Load : Yüklenecek javascript dosyaları noktalı virgül ile ayrılmış verilir.

3.	PageLoadEvents : Sayfa pageload’a olayında çalışacak metotlar verilir. 3 tipi vardır. AfterPageLoad, AfterControlsCreatePageLoad ve BeforeBindControlsPageLoad dır.  AfterPageLoad, sayfa tamamen yüklendikten sonra , AfterControlsCreatePageLoad, kontroller oluştrululduktan sonra, BeforeBindControlsPageLoad, koıntroller bind edilmeden önce çalışır.
  * Type : Collection’ın assembly name’i verilir. Zorunludur.
  * MethodName : Çalışacak metot adı yazılır. Zorunludur.
  * StrPrm : Parametre adı verilir. 

4.	event : Javascript’den c# kodu çağırmak için kullanılan bir yapıdır.
  * eventID : Event’ın adıdır. Zorunludur.
  * jsFunctionName : c# kodu çalıştıktan sonra geri dönen değer bu fonksiyona gönderilir. Zorunludur.
  * jsErrorFunctionName : Kod çalıştırılırken bir hata oluşur ise bu fonksiyona düşer.
  * Type : Çalıştırılacak objenin adıdır.
  * MethodName : Çalıştırılacak objenin metot adıdır.

5.	menuitem : Popup menü eklemek için kullanılır.
  *	name : Popup menünün adı. Zorunludur.
  *	text : Popup menünün caption’ı. Zorunludur.
  *	ControlName : Hangi kontrolde çıkacağını gösterir. ControlName Form olabilir veya gridlerden herhangi birinin adı olabilir. Zorunludur.
  *	jsFunctionName : Çalışacak javascript fonksiyon adı. Zorunludur.
  *	VisibleProcessType : Ekranın hangi modunda görüleceğini söyler. Processtype’lar (New,Update,Delete,Copy,Analyze,MainList,SelectionList,MultiSelectionList,SaveCard,History,OnlyDetail) dir. Noktalı virgül ile ayrılarak yazılır.
  *	OnPopUp : Popup açılırken javascript fonksiyonu vermek için kullanılır.
6.	LayoutUpButtons : tabcontrol’ün üstünde section açmak için kullanılır.
  *	ColumnCount : section’ın kaça bölüneceğini belirtilir. Daha sonra row ve cell eklenir.

7.	LayoutDownButtons : tabcontrol’ün altında section açmak için kullanılır.
  *	ColumnCount : section’ın kaça bölüneceğini belirtilir. Daha sonra row ve cell eklenir.

8.	tabcontrol : Ekrana bir tane tabcontrol ekler.

9.	tabpage : Tabcontrol’un içerisine bir tane tabpage ekler.
  *	Caption : Tabpage’in başlığı belirtilir.
  *	CaptionVisibility : Başlığın gözükmeyeceğini belirtir.
  *	Id : tabpage’e id vermek için kullanılır.

10.	section : Gruplama yapmak için ekrana bir tane tablo ekler.
  *	Visibility : tablonun altına çizik çizmeyeceğine karar verir.
  *	Caption : tablonun başlığını belirtilir.
  *	CaptionVisibility : Başlığın gözükmeyeceğini belirtir.
  *	CaptionVisibility : Başlığın gözükmeyeceğini belirtir.
  *	Id : tabpage’e id vermek için kullanılır.
  *	HtmlStyle : Tabloya style ekler. Noktali virgül ile ayrılır.
  *	CallbackPanelId : tabloları bir panel içine koyar. Birden fazla gridi bu şekilde koyarak tek seferde performcallback yapabilirirz.

11.	 row : tabloya satır ekler.
  *	HtmlStyle : Satıra style ekler. Noktali virgül ile ayrılır.

12.	cell : Satırlara hücre ekler.
  *	colspan : Hücreleri yatayda birleştirmek için kullanılır.
  *	rowspan : Hücreleri dikeyde birleştirmek için kullanılır.
  *	HtmlStyle : Hücreye style ekler. Noktali virgül ile ayrılır.
  *	Id : Hücreye id verir. 

13.	control : Kontrol eklemek için kullanılır.
  *	FieldName : Kontrolun field adıdır. Objede olmak zorunda değildir.  Zorunludur.
  *	ControlType Hangi tip kontrol create edileceğini belirtir. Kontrol tipleri TextEdit, SpinEdit, MemoEdit, ComboEdit, ListEdit, DateEdit, RadioButtonList, CheckEdit, ButtonEdit, Button, Label, HiddenEdit, GridEdit, UploadControl, ColorEdit, BinaryImage, TreeList, ProgressBar, CallbackPanel, HTMLEditor, UserControl, ChartControl, LinkEdit dir. Zorunludur.
  *	Caption : Kontrolün başlığı.
  *	CaptionSize : Kontrolün başlığının genişliği.
  *	ToolTip :  Tooltip verilir.
  *	ControlRequired : Zorunlu olduğunu belirtir.
  *	ControlEnabled : Disable yapmak için kullanılır.
  *	ControlEditEnabled : Düzelt modunda kontrolün değiştirilmemesini sağlar.
  *	ControlVisible : Kontrolü gizlemeyi sağlar.
  *	CaptionVisible : Başlığı gizlemeyi sağlar.
  *	Width : Kontrol genişliğini belirtir.
  *	Height : Kontrol yüksekliğini belirtir.
  *	MaxLength : Maximum uzunluğu belirtir. Text alanda varsayılan 20’dir.
  *	DefaultValue : Yeni modunda varsayılan değer verir.
  *	RegEx : RegEx verir.
  *	ErrorText : Hata olunca hatayı gösterir.
  *	ServerAttribute : Kontrolün değerlerini reflection ile set etmeye yarar. Noktalı virgül ile ayrılır. 
  *	Focused : Ekranda ilk önce hangi kontrole focus olacağını belirtir. Yazılmamış ise en üst kontrole focus olur.
  *	NotFirstNew : Grid ilk açılınca yeni modunda açılmamasını sağlar.
  *	VisibleProcessTypes : Ekranın hangi modunda kontrolün gözükeceğini belirtir.
  *	Password : Kontrolün şifreli gözükmesini sağlar.
  *	CommandName : Komut yetkisi yok ise kontrol gözükmez.
  *	DecimalPlaces : Virgülden sonraki basamak sayısı belirlenir. Hazır kalıplar vardır. Kur=6, BirimFiyat=8,Tutar=2,Miktar=5,Sıfır=0,Oran=3,Pul=4 veya 1,2,3…20 yazabilirsiniz.
  *	Minus : Eksi değere izin vermek için kullanılır.
  *	Editing : ButtonEdit alanına yazı yazmayı engellemek için kullanılır.
  *	Incomplate : ButtonEdit alana yazı yazdığınızda yazdığınız yazı kalır. Arka tarafta sorgu yapıp tamamlama yapmaz.
  *	DisplayFormat : Format vermek için kullanılır.
  *	LoadURL :  User kontrole dosya yüklemek için kullanılır.
  *	ReleatedProperty : Bir kontrolün bağlı olduğu kontrolü belirler. O kontrol değiştiğinde bağlı kontrol silinir.
  *	GroupSummary : Gridde gruplama yapmak için kullanılır. Sum, Min, Max, Count, Average, Custom, None değerleri alır.
  *	GroupSummaryLabel : Gruplama yaptıktan sonra başlığı belirlenir.
  *	TabIndex : TabIndex vermek için kullanılır.
  *	Mask : Mask vermek için kullanılır. Devexpresin mask yapısı kullanılmıştır.
  *	Wrap : MemoEdit’e Wrap özelliği ekler. 

14.	DataSource : Kontrole datasource eklemek için kullanılır. Aşağıdaki kontrollerin datasource’u vardır.

  *	ComboEdit, ListEdit, RadioButtonList
    -	SourceType : Değerler Enum, TextAndValue, ObjectCollection olabilir. Zorunludur. 
    -	Source : Enum için enum değer, TextAndValue için boş, ObjectCollection için collection name olmalıdır. Zorunludur.
    -	Filter :  ObjectCollection için filtrelemede kullanılır.
    -	FilterValues :  ObjectCollection için filtrelemede kullanılır.
    -	OrderByProperty : Sıralama için kullanılır.
    -	TextProperty : ObjectCollection için combo’da gözükecek text property’dir.
    -	ValueProperty : ObjectCollection için combo’da gözükecek value property’dir.
    -	RelatedProperty : Burada bağlı olan başka combobox ve listbox’ın fieldname’i yazılır. Bağlı combo değiştiği zaman bu combo tetiklenir.
    -	EnumValues : SourceType enum seçildiği zaman. EnumValues’de hangi değerler yazıldı ise sadece onlar gösterilir. Boş geçilirse tüm değerler gösterilir. Değerler integer ve araya noktalı virgül ile yazılır.
    -	Text : SourceType TextAndValue seçildiği zaman comboya yazılacak text değerler noktalı virgül ile yazılır.
    -	Value : SourceType TextAndValue seçildiği zaman comboya yazılacak value  değerler noktalı virgül ile yazılır.

  *	ButtonEdit
    -	SourceType : Değerler Command olabilir. Zorunludur.  
    -	Source : Komut adı olmalıdır. Zorunludur.
    -	SourceNumber : Birden fazla datasource bağlandığı zaman 00 dan başlayarak numara verilir.
    -	SourceKeyValue : Birden fazla datasource bağlandığı zaman hangi datasource kullanacağını RelatedProperty deki controlun değerini okuyarak, burda yazan değerle karşılaştırarak karar verir.
    -	Filter :  Filtrelemede kullanılır. Ayrıntısı ayrıca açıklanmıştır.
    -	FilterValues :  Filtrelemede kullanılır. Ayrıntısı ayrıca açıklanmıştır.
    -	OrderByProperty : Sıralama için kullanılır.
    -	ReturnProperties : Açılan object’deki geri dönecek property’ler noktalı virgül ile yazılır.
    -	ReturnedProperties : Açılan object’deki geri dönecek property’ler hangi controllere set edilecek ise noktalı virgül ile yazılır.
    -	ListPropertyName : 1 ise tek seçim 2 ise çoklu seçim ekranı açar.
    -	CustomOpenJs :Ekran açılmadan önce çalışacak javascript fonksiyondur. Ayrıntısı ayrı yazılmıştır.
    -	CustomReturnJs :Seçim yapıldıktan sonra çalışacak javascript fonksiyondur. Ayrıntısı ayrı yazılmıştır.
    -	RelatedProperty : Birden fazla datasource verileceği zaman datasource’un bağlı olduğu kontrolün adı yazılır. 


  *	GridEdit
    -	SourceType : Objectcollection sabiti olmalıdır.
    -	Source : Objectcollection adı olmalıdır.
    -	Filter :  Filtrelemede kullanılır. Ayrıntısı ayrıca açıklanmıştır.
    -	FilterValues :  Filtrelemede kullanılır. Ayrıntısı ayrıca açıklanmıştır.
    -	CloseDetailCallback : Seçilen satır değiştiği zaman bu gride bağlı detayların callback’ı çağrılmaz.
    -	CloseAutoNew : Grid otomatik yeni modunda açılmamasını sağlar.
    -	CopyDetailGrid : Grid copy yapıldığı zaman buna bağlı hangi gridlerinden kopyalanması isteniyor ise arada noktalı virgül koyularak grid adları yazılır.
    -	IgnoreUpdateProperty : Grid update edilince bu property’ler update edilmez. Noktalı virgül ile yazılırlar.
    -	FirstExcelColumnList : Gridün üzerinde excelden yapıştır yapıldığı zaman hangi alanların önce işlem görmesini istiyorsanız. Bu property’ler noktalı virgül ile ayrılarak yazılır.
  
  *	ChartControl
    -	View : Değerler devexpress’in view tipleridir.  

15.	ClientSideEvents : Kontrole client side event ekleme yapmak için kullanılır. Devexpress’in eventları kullanılır. Client Side eventları görmek için devexpress’in help’inde arattırma yapmak için aspx’den sonra client yazınız. Örnek AspxClientTextBox gibi…. 
En çok kullanılan eventlar aşağıdadır.
  *	ValueChanged: Kontrolun değeri değiştiği zaman çalışır.
  *	EndCallback: Gridde callback işlemi tamamlanınca çalışır.
  *	Click: Button’a clickleme yapınca çalışır. 

16.	hidden : Gizli kontrol hiddenedit eklemek için kullanılır.



##	STANDART EKRANLAR NASIL YAPILIR?

Yeni, sil ve düzelt yapılabilen, ana ekran açılmadan önce bir liste ekranı açılan, kayıt işlemi ve controllere bind işlemi otomatik yapılan ekranlara standart ekran diyoruz. Standart ekranlar GeneralCard.aspx dosyası ile yüklenirler.

*	Object Oluşturma: Object’nin ne olduğu 2. soruda anlatılmıştır. Öncelikle orada tarif edildiği gibi object ve collection’ınımızı oluştururuz. Oluşturulan sınıflarımızı ilgili modüldeki dll’imize ekliyoruz. DLL’e ekledikten sonra UyumCreateOrUpdate çalıştırılarak database’e bu tablo açılmış olur. UyumCreateOrUpdate tanımlamak için, Visual studio’nun menüsünden Tools -> Externel Tools seçilir. Aşağıdaki ekranda görüldüğü gibi tanım yapılarak Add tuşuna basılır.

 

Command : D:\UyumProjects\Senfoni\trunk\Uyum.Developer.Tools\bin\Debug\ Uyum.Developer.Tools.exe
Arguments : $(ItemPath)
Inıtial directory : D:\UyumProjects\Senfoni\trunk\Uyum.Developer.Tools\bin\Debug\

*	Komut Oluşturma : Ekranların açılma işlemleri komutlar ile olmaktadır. Kart ekranlarında standart olarak  Göster, Yeni , Düzelt, Sil, Kopyala, İncele, Log komutları oluşmaktadır. Komut tanımlama ekranı senfoni projesinin içerisinde Kurumsal Kaynak Yönetimi -> Yazılım Geliştirme Test -> Komut Tanımı ekranıdır. Aşağıdaki gibi tanımlanır.

 


*	XML Oluşturma : Ekranlar xml tasarlayarak oluşturulmaktadır. XML’in ne olduğu ve özellikleri 3. Soruda anlatılmaktadır. XML’i orada anlatılan şekilde oluşturduktan sonra Senfoni projesinin altındaki ilgili modüldeki klasöre eklemek gerekmektedir. 

Bu adımları tamamladıktan sonra ekranlar çalışır duruma gelmektedir.

##	STANDART OLMAYAN EKRANLAR NASIL YAPILIR?

Bu ekranlara service ekranları da denmektedir. XMLCard.aspx dosyası ile yüklenirler. Ekranın bir objesi ve standart kayıdı yoktur. Sadece ekranın XML’i vardır ve altyapı bu XML’i generate ederek ekranı oluşturur. Diğer yapılacak işlemlere yazılımcı karar verir. Ekranın komutu oluşturulurken CollectionName boş bırakılır, ListPageType  XMLCard.aspx seçilir ve sadece Show Command Ekle seçeneği seçilir. 


##	LİSTE EKRANINDA KOLON SIRASINI, SIRALAMAYI NASIL VEREBİRİM?

APPD_FILTER tablosuna object_type alanına collection’ın assembly name’i yazılır. ListBrowsableString alanına propertyname’ler araya virgül bırakarak yazılır. Eğer özellikle uzunluk verilecek ise : yazılarak uzunluk verilir. Virgülden sonra gösterilecek basamak sayısını belirtmek için ise tekrar : yazıldıktan sonra basamak sayısı yazılır.

Sıralama için Order_by filedname’ine sıralanacak alanların propertyname’ leri  araya virgül konarak yazılır.
 
Örnek : ToolGroupMCode:30,AmtTra:200:2,Ispassive:10

##	LİSTE EKRANINDA FILTER NASIL VEREBİRİM?

APPD_FILTER tablosuna object_type alanına collection’ın assembly name’i yazılır. Filter_String alanına xml adı yazılır.

Örnek : FIN/FILTER/AccFilter.xml

##	APPD_FILTERDAKİ ALANLARIN ÖZELLİKLERİ NEDİR? 
AppdFilter liste ekranının bilgilerini tutar.

ObjectType : Collection’ın aseembly name’i.
FilterString   : Liste üzerinde custom filter vermek için kullanılır.
ListBrowsableString :  Listede gözükecek kolon listesi verilir.
OrderBy : Listenin sıralanacağı alanı verir.
RowCount : Gösterilecek satır sayısını verir.
MainListName : Ekranın ismidir.
Version : Aynı objenin birden fazla listesi var ise bu alan kullanılır.
HiddenField : Listede sürekli olması gereken alanlar için kullanılır. 
   

##	BİR KONTROLDE REGULER EXPRESSION İLE NASIL VALIDATION YAPABİLİRİM? 

AppdFilter liste ekranının bilgilerini tutar. Kontrolun RegEx attribute’ini kullanarak regex set edebiliriz. Validation sonuncunda çıkacak hatayı da ErrorText attribute’in içerisine yazabiliriz. 

Aşağıdaki örnek mail kontrolü yapmaktadır.
```xml
<control FieldName = "Email" ControlType = "TextEdit" Caption = "Email" ControlRequired = "true" ControlEnabled = "true" ControlVisible = "True" ControlSingleLine = "true" RegEx="\w+([-+.']\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*" ErrorText="Hatalı Giriş Fotmatı"> 
```
Regular Expression’larda Kullanılan Özel Karakterler ve Etkileri
Regular expression desenleri tanımlamada kullanılan özel karakterleri örnekleri ile anlatırsak sanırım regular expressionlar daha tanıdı ve kolay gelebilir.
1.  **.** Karakteri
Tek bir karakteri temsil eder(yeni satır karakteri hariç).
CSharp.edir şeklindeki bir desen CSharpnedir, CSharpNedir, CSharpSedir, CSharp3edir gibi stringleri döndürebilir.
2.  **[]** Karakterleri
Bir arrayi yada aralığı temsil eder.
CSharp[SNY]edir deseni, CSharpSedir, CSharpNedir ve CSharpYedir stringlerini döndürür.
CSharp[a-z]edir şeklindeki kullanım aralık belirtmeye yarar.
CSharp[0-9]edir şeklindeki kılanlım ise sayısal aralık belirtmeye yarar.
3.  **?** Karakteri
Kendinden önceki karakterin stringte olması yada olmamasını sağlar.
CSharpn?edir deseni CSharpedir yada CSharpnedir döndürür.
4.  __\\__ Karakteri
Kendinden sonraki özel karakterin stringe dahil edilmesini sağlar.
CSharpnedir\? deseni CSharpnedir? Stringini döndürür. (Eğer \ karakterini kullanmamış olsaydık CSharpnedi yada CSharpnedir dönerdi.)
5.  __*__ Karakteri
Kendinden önceki karakterin yada stringin hiç olmaması yada istediği sayıda olmasını sağlar.
CSharpnedir* deseni, CSharpnedi, CSharpnedir, CSharpnedirr, CSharpnedirrr, ... döndürür. CSharp(nedir)* deseni ise CSharp, CSharpnedir, CSharpnedirnedir, ... döndürür.
6.  **{}** Karakterleri
Kendinden önce gelen karakterin belirtilen sayıda tekrar etmesini sağlar.
C{4}Sharpnedir deseni, CCCCSharpnedir stringini döndürür.
7.  **^** Karakteri
Satır başını ifade eder.
^CSharpnedir deseni, satır başında CSharpnedir stringi varsa bunu döndürür.
8.  **$** Karakteri
Satır sonunu ifade eder.
CSharpnedir$ deseni, satır sonunda CSharpnedir stringi varsa bunu döndürür.



##	GRİDDE GÖSTERİLECEK SATIR SAYISINI NASIL DEĞİŞTİREBİLİRİM?
AppdFilter Giridinizde bir sayfada görülecek kayıt sayısını aşağıdaki gibi ayarlayabilirsiniz...
ServerAttribute="SettingsPager-PageSize=20"
Giridinizde bir sayfada tüm kayıtları göstermek istiyorsanız aşağıdaki gibi ayarlayabilirsiniz.
ServerAttribute="SettingsPager-Mode=ShowAllRecords "

##	SAYFA YÜKLENİRKEN HANGİ MODDA OLDUĞUNU NASIL ALIRIM? 
Page nesnesini BasePage’e cast ederseniz, içerisinde bulunan ProcessType property’si ile anlayabilirsiniz.
Örnek :
```cs
  BasePage basePage = (page as BasePage);
  if (basePage.ProcessType == ProcessTypes.Update)
  {
    //Kontrol kodunu buraya yazabilirsiniz.
  }
```
##	 SAYFANIN İLK AÇILIŞINDA CLIENTSIDE’DA JAVASCRIPT FONKSIYONU NASIL ÇAĞIRABİLİRİM?  

AddLoadList Javascript fonksiyonunun içerisine kendi fonksiyonunu yazarak çağırabilirsiniz..

Örnek :
```cs
   AddLoadList('FirstLoad()');   
   FirstLoad=function() 
   {
      alert(‘a’);
   }
```
##	RAPORLARA CARİ, STOK, BANKA KRİTERLERİNİ NASIL EKLERİM ? 

AppdFilter liste ekranının bilgilerini tutar. Kontrolun RegEx attribute’ini kullanarak regex set edebiliriz. Validation sonuncunda çıkacak hatayı da ErrorText attribute’in içerisine yazabiliriz. 
                Cari,Stok ve Banka Rapor kriterleri için altyapı çalışması yapılmıştır. Rapor kriterlerini eklemek istediğiniz ekranda aşağıdaki adımları sırasıyla yapınız…

1.	Ekranınıza bir tane buton  yerleştiriniz ve butona bir tane isim veriniz. (Cari Rapor Kriterleri, Stok Rapor Kriterleri, Banka Rapor Kriterleri)
 

2.	 Butona aşağıdaki gibi ClientSideEvent yazınız. 
```xml
<control FieldName="BtnReporParameters" ControlType="Button" Caption="Cari Rapor Kriterleri">
                <ClientSideEvents Click="function(s,e)
                 {
                    OpenReportExtra('Entity');
                 }">
                </ClientSideEvents>
              </control>
```
OpenReportExtra parametresine Cari için ('Entity'),Stok için (‘Item'), Banka için (‘BankAcc') yazınız.

3.	C# kod kısmı için 
  *	Öncelikle kodun en üst satırına string currentIdentity = ur._getDataTableValues(prms, "CurrentIdentity").Split('|')[0]; eklenir.
  *	Filtre eklemek istediğiniz objenize 
Cari için PageReportsCode.AddEntityReport,
Stok için PageReportsCode. AddItemReport,
Banka için PageReportsCode. AddBankAccReport static methodu yazılır. Methodların parametreleri aşağıdaki gibidir.

Örn : PageReportsCode.AddEntityReport(typeof(InvoiceM), manInvoiceMaster, currentIdentity);

Bu kod satırı man objesine exist olarak eklenerek sadece ilgili carilerin gelmesini sağlayacaktır.
            
      Örnek Ekranlar için;
**Satış/Satınalma yönetimi/** Raporların altında **Fatura ve Fatura kalemleri** listesine bakılabilir.
      Örnek Cari Kriter ekranı :
 


##	OBJELERDE STANDART LOAD ,İŞLEMİ NASIL YAPILIR?
 
Objelere databaseden kayıt yükleme ObjectLoadManifest ile yapılır. 
Örnek :
```cs
using Uyum.Objects;
using GNL;

//Instance’ı yaratılır.
ObjectLoadManifest manifest = new ObjectLoadManifest();
//koşul verilecek ise bir tane SqlConditionManager yaratılır.
manifest.Filter.Conditions=new SqlConditionManager();
//SqlOperratorleri ile koşulk verilir.
manifest.Filter.Conditions.Add(new SqlOperatorEquality("GNLD_COMPANY.CO_ID",coId));
//Tüm alanlar çekilmeyecek ise çekilecek alanlar verilir.
manifest.SelectProperties.Add("StartDate");
manifest.SelectProperties.Add("CurId");
//sıralama var ise sıralama verilir.
manifest.OrderBys.Add("EntityCode");
//veri yüklenir
CompanyCollection companyCol = GlobalObj.Load< CompanyCollection >(manifest);
```



##	OBJELERDE LOAD İŞLEMİNİ KISA YOLDAN NASIL YAPABİLİRİM ? 

Objedeki Load işlemlerin daha kolay yapılması için bir yeni iki tane static Generic method eklenmiştir. Bu methodlar artırılacaktır. Data göndererek veya göndermeden objeyi tek satırda yükleyebilirsiniz.
Parametre olarak collectionın tipini ve sqlconditionmanager oluşturmak için dictionary ile PropertyName ve Value’sunu geçiyorsunuz…  Böylelikle tek satırda yükleme yapabiliyorsunuz.
1.	Data Göndermeden
```cs
TownCollection townCol2 = GlobalObj.Load<TownCollection>(new Dictionary<string, object>() { { "TownName", "ÇAYELİ" } });
```
veya
```cs
string town = "ÇAYELİ";
TownCollection townCol = GlobalObj.Load<TownCollection>(new Dictionary<string, object>() { { "TownName", town } } );
veya
TownCollection townCol3 = GlobalObj.Load<TownCollection>(new Dictionary<string, object>() { { "TownName", "ÇAYELİ" }, { "CityId", "11574" } });
veya 
Dictionary<string, object> dic = new Dictionary<string, object>();
dic.Add("TownName","ÇAYELİ");
TownCollection townCol3 = GlobalObj.Load<TownCollection>(dic);
```
2.	Data Göndererek
```cs
TownCollection townCol2 = GlobalObj.Load<TownCollection>(data, new Dictionary<string, object>() { { "TownName", "ÇAYELİ" } });
```


##	BUTTONEDİT’LERDE GERİ DÖNÜŞ DEĞERLERİNİ KENDİM YÖNETEBİLİRMİYİM?

ButtonEdit’lerin ButtonClick’ini kendiniz yönetebilirsiniz. İstediğiniz bir sayfayı, istediğiniz parametrelerde açabilirsiniz. Geri döndürdüğü değerleri yine de otomatik set edebilirsiniz. Bunun için yeni bir attribute geldi.   CustomOpenJs bu attribute’te buttonclick’e tıklandığı zaman gitmesini istediğiniz javascript fonksiyonunu yazın. Javascript fonksiyonu 3 parametre almaktadır. Control, listUrl ve args’tır.
Örnek Kod :
```js
function CustomOpenCityList(comp,listUrl,args)
{
  return window.showModalDialog(listUrl + "&Args='" + args + "'" , self, "dialogHeight:"+800+"px;dialogWidth:"+600+"px;");
}
```
```xml
<control FieldName="CountryName" ControlType="ButtonEdit" Caption="Ülke" ControlRequired="true">
<DataSource SourceType="Command" Source="CountryCollection.Show" ProcessTypeMode="1" Filter="" FilterValues="" ReturnProperties="Id;PhoneCode;CountryName" ReturnedProperties="CountryId;PhoneCode;CountryName" ListPropertyName="CountryName" CustomOpenJs="CustomOpenCityList">
  </DataSource>
</control>
```

##	GRİDLERDEKİ KOLONLARA HYPERLINK NASIL EKLEYEBİLİRİM?

Gridlere HyperLink kontrolu ekleme özelliği eklendi. Bunun için ControlType=="LinkEdit" vermeniz gerekiyor. Ayrıca NavigateUrlFormatString ve TextField attributeleri. Bunlar tıklandığında gidecek url adresinin göstermektedir. NavigateUrlFormatString içindeki {0} değerinin yerine TextField belirtiğiniz fieldname gelecektir.

```xml
<GridColumn FieldName="LineNo" ControlType="LinkEdit" NavigateUrlFormatString="="~/FileUploads/{0}" TextField="ShortFileName"></GridColumn>
```
Griddde aşağıdaki gibi bir sonuç verir.
```xml
<dxwgv:GridViewDataHyperLinkColumn Caption="Dosya Adı" 
                            FieldName="LongFileName" Name="LongFileName" VisibleIndex="3" 
                            Width="100px">
                            <PropertiesHyperLinkEdit NavigateUrlFormatString="~/FileUploads/{0}" 
                                TextField="ShortFileName">
                            </PropertiesHyperLinkEdit>
```

Örnek Ekran Görüntüsü :

 


##	BUTTONEDİT’LERDE BAŞKA BİR KONTROLÜN DEĞERİNE GÖRE TEKLİ SEÇİM VEYA ÇOKLU SEÇİM AÇTIRABİLİRİM?
                ButtonEdit açılırken bağlı bulunduğu bir alana göre bazen multi select moda bazende tekli seçim modunda açılabilmesini sağlamak için, ButtonEdit alanın DataSource’una yeni bir property eklenmiştir.  RelatedProcessTypeName , bu property ilgili alanın vermeniz gereklidir. Bu alanın değeri 7 ise multi select modunda diğer durumlarda sadece select moda mainlistti açmaktadır.

Örnek :
```xml
<cell>
  <control FieldName="Operation_BranchCode" ControlType="ComboEdit" Caption="İşyeri Kodu" Width="75" DefaultValue="6">
    <DataSource SourceType="Enum" Source="Operator, UYM, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null">
    </DataSource>
  </control>
</cell>
<cell>
  <control FieldName="BranchCode_1" DefaultValue="Session(BranchCode)" ControlType="ButtonEdit" CaptionVisible="false" Width="100">
    <DataSource SourceType="Command" Source="BranchCollection.Show" ReturnProperties="BranchCode;CoId" ReturnedProperties="BranchCode_1;Operation_CoId" OrderByProperty="" ListPropertyName="BranchCode" ProcessTypeMode="3" RelatedProcessTypeName="Operation_BranchCode" />
  </control>
</cell>
```

##	MAİL GÖNDERME İŞLEMİ NASIL YAPABİLİRİM? 

1.	Javascriptle e-mail göndermek için
```js
var mailForm = new MailForm();
              mailForm.To = 'harun.urut@uyumsoft.com.tr;bulent.sari@uyumsoft.com.tr';
              mailForm.CC = 'bulent@bilgiparki.com';
              mailForm.Subject = 'Sipariş Formu';
              mailForm.Body = '10.01.2010 tarihli iğüşçö Nolu Sipariş Formunuz.';
              mailForm.Attach = 'c:\\deneme.txt';
              var modaldialog = SendMailDialog(mailForm);
```
2.	C# ile e-mail göndermek için
```cs
 GlobalClass.SendMail(string emailUserName, string displayName, string mailServerAddress, int port, string password, string[] receiveAddresses, string[] CCAddresses, string subject, string body, string[] atachFileNames);
 ```
Adresini kullanabilirsiniz.
    Kişilerin  mail adres bilgileri kullanıcı adı, görünen ad, şifre ve  serveradı gibi bilgileri Kullanıcı işlemlerin altındaki, Kullanıcı e-posta tanımlarında bulunmaktadır.
    
     Mail gönderme Ekranı

 

Kullanıcı mail tanıtma ekranı

 


##	BUTTON EDİTLERE BİRDEN FAZLA DATASOURCE’UNU MASTERDAKİ ALANA GÖRE NASIL YÖNETİLİR ?

Button Editlere Birden Fazla DataSource Vererek Başka Bir Alandan Gelen Tipe Göre Farklı Seçim Ekranları Verebiliyorduk .

Fakat Gridlerde İlişkili Olacak Alan Sadece Grid Üzerinde ki Bir Alan Olabiliyordu . Yeni Özellik Sayesinde Ekrandaki Bir Master alan Üzerinden Data Source Seçimi Yapabiliriz. Tek Yapmamız Gereken RelatedProperty Tagına Veri Girerken Veri Alacağımız Alan Master üzerindeyse M(PropertyName) Şeklinde Girmek Olacaktır ..
SourceKeyValue değerine bakarak hangi datasource’un kullanılacağını belirler
SourceNumber, 00 ile başlar 01,02 … diye her datasource için artar.

Örnek;
```xml
  <DataSource SourceType="ObjectCollection" SourceNumber ="00" Source="FIN.TraTypeCollection, FIN, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" SourceKeyValue ="3"  OrderByProperty="TraTypeCode" Filter="And;SourceApp=@SourceApp;IsBank=@IsBank" FilterValues="@SourceApp=10;@IsBank=1" ReturnProperties = "CardType;Id;PlusMinus;Description;TraTypeCode" ReturnedProperties = "CardType;TraTypeId;PlusMinus;Description;TraTypeCode" RelatedProperty="M(CardType)" ListPropertyName="TraTypeCode" ></DataSource>
<DataSource SourceType="ObjectCollection" SourceNumber ="01" Source="FIN.TraTypeCollection, FIN, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" SourceKeyValue ="2"  OrderByProperty="TraTypeCode" Filter="And;SourceApp=@SourceApp;IsBank=@IsBank" FilterValues="@SourceApp=10;@IsBank=1" ReturnProperties = "CardType;Id;PlusMinus;Description;TraTypeCode" ReturnedProperties = "CardType;TraTypeId;PlusMinus;Description;TraTypeCode" RelatedProperty="M(CardType)" ListPropertyName="TraTypeCode" ></DataSource>
```
##	BUTTON EDİT ALANLARINDA ENUM DEĞERİNİN VALUE’SUNU NASIL ALABİLİRİM?

ButtonEditler'le açtığınız liste ekranda, Enum bir alanın value değerinin döndürmek isterseniz (Yani sayısal değeri) ReturnProperties alanına fieldname'in başına @eklemeniz yeterli olacaktır.
CardType ı almak istiyorsanız ReturnProperties="@CardType" ReturnedProperties="CardType" şeklinde yazmanız gerekmektedir. 
 Not : ReturnedProperties CardType aynısı gibi kalması gerekmektedir.
 Örnek :
 ```xml
<control FieldName = "ReceiptTypeCode" ControlType = "ButtonEdit" Caption = "Fi Tipi" ControlRequired = "true" ControlEnabled = "true" ControlVisible = "True" ControlSingleLine = "true" Focused ="True">
<DataSource SourceType="ObjectCollection" Source="FIN.ReceiptTypeCollection, FIN, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" Filter="" FilterValues="" OrderByProperty="ReceiptTypeCode" ReturnProperties="Id;ReceiptTypeCode;@CardType;Description" ReturnedProperties="ReceiptTypeId;ReceiptTypeCode;CardType;Description" RelatedProperty="">
</DataSource></control>
```

22)	KONTROLLERİN LABEL’LARINI NASIL DEĞİŞTİREBİLİRİM ?

Ekrandaki labellara(caption) c# tarafından erişip değişiklik yapabilirsiniz veya javascript tarafından da erişebilirsiniz.
 c# tarafından labellara erişmek için;
 AfterControlsCreatePageLoad eventi yazmanız gerekmektedir.
  Firma Kodunun fieldname'i CoCode ise label adı "lblCoCode" olarak create edilmektedir. 
 Erişmek için aşağıda örnek verilmiştir.
 Örnek :
 ```xml
<PageLoadEvents>
<AfterControlsCreatePageLoad Type="Senfoni.ABT.AbtGlobalClass,Senfoni,Version=1.0.0.0,Culture=neutral,PublicKeyToken=null" MethodName="AfterControlsCreatePageLoadRequest">
</AfterControlsCreatePageLoad>
<AfterPageLoad Type="Senfoni.ABT.AbtGlobalClass,Senfoni,Version=1.0.0.0,Culture=neutral,PublicKeyToken=null" MethodName="AfterPageLoadCalcTotalAmountAndCount" StrPrm="RequestDCol|AMT_TRA">
</AfterPageLoad>
</PageLoadEvents>
```
```cs
 Label lbl = (Label)GlobalObj.gc.GetControl(CurrentPage, "lblCoCode");
lbl.Text = "aaa bbb";
```
 Javascriptten erişmek için;
 ```js
GetControlValue("lblCoCode") 
SetControlValue("lblCoCode")
```
 kullanabilirsiniz...
.
23)	TABLODAKİ BİR ALANDA BİR DOSYAYI NASIL SAKLAYABİLİRİM ?

    Bir kayıda resim veya herhangi bir dosya ekleyebilirsiniz. Bu dosya veritabanında saklanmaktadır..
Javascipt kodu :
 ```xml
<root MainCode="GNL1001" Caption="Firma TanIm">
<script ScriptCode ="
function OnUploadStart() {
btn_btnUpload.SetText('Dosya Yükleniyor...'');
btn_btnUpload.SetEnabled(false);
}
function OnUploadComplete(args) {
btn_btnUpload.SetText('Dosya Yüklendi''); 
btn_btnUpload.SetEnabled(true);
}
"></script>
```
 *XML tag :*
 ```xml
 <row>
<cell colspan="2">
<control FieldName = "Image1" CaptionVisible="False" Width="250" ControlType = "UploadControl">
<ClientSideEvents
FileUploadComplete="function(s, e) { OnUploadComplete(e); }"
FileUploadStart="function(s, e) { OnUploadStart(); }">
</ClientSideEvents>
</control>
</cell>
<cell colspan="1">
<control FieldName = "btnUpload" ControlType = "Button" CaptionVisible="False" Caption="Ykle">
<ClientSideEvents Click="function(s, e) { upl_Image1.UploadFile(); }" />
</control>
</cell>
</row>
```
**Objedeki alanın property’si byte[] olmalıdır..**


##	GRİDDEKİ DEĞERLERİ SESSIONDAN OKUMAK ?
```cs
  BindingList<FinD> list =  Session[typeof(FinD).GetHashCode().ToString()+'.'+(page as BasePage).CurrentIdentity]  
```
##	EKRANA UPDATEPANEL KONTROLU NASIL EKLERİM ?
XML ekranlarında UpdatePanel kontrolu create edebilirsiniz… Bunun için  ControlType=CallbackPanel attribute’u yazmanız yeterli olacaktır. Javascript ile kontrolun PerformCallback’ini çağırarak ve c# tarafında callback eventini yakalayarak runtime da ekrana yeni kontroller create edebilirsiniz…
      
Örnek :
```xml
<control FieldName="RegisterAddressCallback" ControlType="CallbackPanel"></control>  
<AfterPageLoad Type="Senfoni.HRM.CustomCode.HrmGnlCodes,Senfoni,Version=1.0.0.0,Culture=neutral,PublicKeyToken=null" MethodName="SetRegisterCallbackPanel">           </AfterPageLoad> 
```      
```cs
public string SetRegisterCallbackPanel(Page page)
{
    ASPxCallbackPanel callBackPanel = gc.GetControl(page, "RegisterAddressCallback") as ASPxCallbackPanel;

    callBackPanel.Callback += new DevExpress.Web.ASPxClasses.CallbackEventHandlerBase(callBackPanel_Callback);

    return string.Empty;
}

void callBackPanel_Callback(object sender, DevExpress.Web.ASPxClasses.CallbackEventArgsBase e)
{
    ASPxCallbackPanel callBackPanel = (sender as ASPxCallbackPanel);
    ASPxTextBox txt = new ASPxTextBox();
    txt.ID = "deneme";
    txt.ClientInstanceName = " deneme ";
    callBackPanel.Controls.Add(txt);
}
```
##	TARİH ALANLARDA TARİH VE SAATİ AYNI ANDA NASIL GÖSTEREBİLİRİM ?
XML Grid’de tarih bilgisini saat ile birlikte girmek ve göstermek istiyorsanız, bunun için yeni bir attribute geliştirildi. Attribute adı DisplayFormat="DateTime"
Örnek :
```xml
<GridColumn FieldName="StartDate" DisplayFormat="DateTime" ControlType="DateEdit"  DefaultValue="Function(GetDateTime)" Width="130" Caption="Başlangıç Tarihi" MaxLength="50"></GridColumn>
```
##	KONTROLU SERVERSIDE’DA  DISABLE YAPMAK?
C# tarafında kontrollerin enabled'larını false yapmak için enabled fonksiyonunun yerine ClientEnabled kullanınız. Aksi halde client tarafında javascript ile değerine ulaşamazsınız.
 Örnek : 
```cs
ASPxTextEdit CurRateTypeCode = GlobalObj.gc.GetControl(page, "CurTraCode") as ASPxTextEdit;
CurRateTypeCode.ClientEnabled = false;
 ```
##	LİSTE EKRANLARINDA VERİTABANINDA TARIH SAAT TUTULAN ALANLARDA SADECE TARİHE GÖRE FILTRE VEREREK NASIUL ÇALIŞTIRABİLİRİM?
Liste ekranlarında tarih filtresi verirken veri tabanında bu alana karşılık saat alanı da var ise sorgular düzgün çalışmamaktaydı. Bu problemin çözümü için altyapı ekibi olarak yeni bir attribute geliştirdik.
Bu attibute’u controle yazarsanız aramalarda saati dikkate almayacaktır. İyi Çalışmalar… 
Attribute : DateFilter="True"
Örnek :
```xml
<row>
  <cell>
    <control FieldName="Operation_StartDate"  ControlType="ComboEdit" Caption="Başlangıç Tarihi" Width="75" DefaultValue="6">
      <DataSource SourceType="Enum" Source="Operator, UYM, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null">
      </DataSource>
    </control>
  </cell>
  <cell>
    <control FieldName="StartDate_1" DateFilter="True" ControlType="DateEdit" CaptionVisible="false" DefaultValue="Function(GetDateTime)" Width="115">
    </control>
  </cell>
  <cell>
    <control FieldName="StartDate_2" ControlType="DateEdit" CaptionVisible="false" DefaultValue="Function(GetDateTime)" Width="115">
    </control>
  </cell>
</row> 
```
##	XML’DE GRİDDEKİ BİR KOLONA GÖRE VARSAYILAN SIRALAMA NASIL EKLERİM?
Kayıt grişi yaptığınız grid'ler için ilk açılışta sırama yaptırabilirsiniz.
Bunun için hangi kolonda sıralama yapmak istiyorsanız GridColumn tagının içine Sort="true" yazmanız yeterli olacaktır.
Örnek :
```xml
<GridColumn FieldName = "DepartmentName" ControlType = "TextEdit" Caption = "Blm Ad" ControlEnabled = "true" ControlVisible = "True" ControlSingleLine = "true" Sort="true"></GridColumn>
```
##	GRİDE FILTRE VEREREK NASIL BAZI KAYITLARIN VERİTABANINDAN ÇEKİLMESİNİ ENGELLERİM?
Artık Kart Ekranlarında ki Detay Gridlerinin İlk Yüklenmesi Esnasında Filtre verebilir Ve İstemediğiniz Detayların Yüklenmesini Engelleyebilirsiniz. Diğer Kontrollere Verdiğimiz Filtre Mantığı İle Aynı Çalışmaktadır. 
Örnek;
```xml
<control FieldName = "TestDepartmentCardCollection" ControlType = "GridEdit" Caption = "Bölümler" ControlRequired = "True" ControlEnabled = "true" ControlVisible = "True" ControlSingleLine = "true">          <DataSource SourceType="ObjectCollection" Source="TEST.TestDepartmentCardCollection, GNL, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" Filter="CountryId=@CountryId" FilterValues="@CountryId>>3"></DataSource>  
```
 Örnekte Görüldüğü Üzere Filtreleyeceğimiz Detay Gridinin DataSource Tagına Filter ve FilterValues Özelliklerini Ekleyerek İstediğiniz Filtre İşlemini Yapabilirsiniz. **Not: Filtre İşlemi Sadece Detaya Bağlı Collection Üzerinde ki Property ler vasıtasıyla yapılabilir.**

##	FILTRE VE FILTER VALUES NASIL KULLANILIR?
  * Eşitlik;
  ```xml Filter="And;CityId=@CityId;TownName=@TownName" FilterValues="@CityId==GetControlValue(CityId);@TownName==Kumru```
  * Eşitsizlik
  ```xml Filter="And;CityId=@CityId;TownName=@TownName" FilterValues="@CityId!=GetControlValue(CityId);@TownName!=Kumru```
  * Büyük
  ```xml Filter="CityId=@CityId " FilterValues="@CityId>>GetControlValue(CityId)```
  * Küçük
  ```xml Filter="CityId=@CityId " FilterValues="@CityId<<GetControlValue(CityId)```
  * Büyük Eşit
  ```xml Filter="CityId=@CityId " FilterValues="@CityId>=GetControlValue(CityId)```
  * Küçük Eşit
  ```xml Filter="CityId=@CityId " FilterValues="@CityId<=GetControlValue(CityId)```
  * StartsWith
  ```xml Filter="TownName=@TownName" FilterValues="TownName%=Kum```
  * EndsWith
  ```xml Filter="TownName=@TownName" FilterValues="TownName%%umr```
  * Contains
  ```xml Filter="TownName=@TownName" FilterValues="TownName=%mru```
  * Between
  ```xml Filter="And;CityId=@CityId; CityId=@CityId2 " FilterValues="@CityId!>>50;@ CityId2<<200```
  * NotBetween
  ```xml Filter="And;CityId=@CityId; CityId=@CityId2 " FilterValues="@CityId!<<50;@ CityId2>>200```
  * In
  ```xml Filter="TownName=@TownName" FilterValues="TownName**Çayeli,Pazar```

##	SERVER SİDE TARAFINDA BİR KONTROLÜN DEĞERİNİ NASIL ALIRIM?
```cs
Object countryValue = GlobalObj.gc.GetControlValue(Page,'Country');
```
##	CLİENT SİDE TARAFINDA KONTROLLERİN DEĞERİNİ OKUMA VE YAZMA?
Javascript tarafından kontrollere ulaşabilmeniz , kontrollerden değer okuyup yazabilmeniz için yeni fonksiyonlar eklenmiştir.
  1. GetControl(FieldName) : Kontrolün kendisine ulaşabilirsiniz... Kontrol devexpress controlu ise client side methodlarından tümünü kullanabilirsiniz.
    Örnek : 
    ```js GetControl('CountryName'').SetValue('Almanya'); // (Button edit için)```
    ```js GetControl('CountryCollection').GetEditor('CountryName').SetValue('Almanya'); //(Grid İçin)```
  2. GetControlValue(FieldName) : Kontrolün içindeki değere ulaşabilirsiniz. Kontrolun devexpress kontrolu veya hidden bir kontrol olması önemli değildir.
    Örnek : ```js GetControlValue('CountryName');```
  3. SetControlValue(FieldName,FieldValue) : Kontrollere değer yazabilirsiniz. Kontrolun devexpress kontrolu veya hidden bir kontrol olması önemli değildir.
    Örnek : ```js GetControlValue('CountryName');```
  4. GetColumnValue(GridName,FieldName) : Kolonun içindeki değere ulaşabilirsiniz. Kolonun visible veya invisible olması önemli değildir.
    Örnek : ```js GetColumnValue('CountryCollection','CountryName');```
  5. SetColumnValue(GridName,FieldName,FieldValue) : Kolonlara değer yazabilirsiniz. Kolonun visible veya invisible olması önemli değildir.
    Örnek : ```js SetColumnValue('CountryCollection','CountryName','Almanya');```

##	SAYFA KAYDEDİLİRKEN OBJEYE GİTMEDEN ARAYA NASIL GİREBİLİRİM?
GeneralKart ekranlarında Kaydet butonuna bastığınız zaman aspx in code tarafında objeye gitmeden önce araya girebilme özelliği eklendi. Araya girerek kaydetmeyi engelleyebilirsiniz (objeye daha gitmeden) veya objede alanların değerlerini değiştirerek kaydedebilirsiniz
**Kullanılışı :**
    XML PageLoad eventi yazılır.  Pageload’da sayfanın BeforeSave eventine event bağlanır. Ve bu eventin içerisinde gelen objeyi değiştirebilirsiniz veya kaydetmeyi iptal edebilirsiniz…
**XML’de :**
  ```xml 
  <PageLoadEvents>
    <AfterPageLoad Type="Senfoni.GNL.GnlCodes,Senfoni,Version=1.0.0.0,Culture=neutral,PublicKeyToken=null" MethodName="CityOnLoad">
    </AfterPageLoad>
  </PageLoadEvents>
  ```
**Code :**
```cs
  public string CityOnLoad(Page page)
  {
      BasePage basePage = (page as BasePage);
      basePage.BeforeSave += new SaveCancelEventHandler(City_BeforeSave);
      return string.Empty;
  }
  void City_BeforeSave(object sender, SaveCancelEventArgs e)
  {
      City cityObj = e.UyumObj as City;
      if (cityObj.PhoneCode.length!=3)
      { 
          e.Cancel = true;
          e.CancelMessage = "Telefon kodu 3 hane olabilir";
      }
  }
```
##	GRİDDE ALT TOPLAM ALIP BUNU BAŞKA KONTROLDE NASIL GÖSTEREBİLİRİM?
    Grid alanlarının alt toplamının alınması ve bu alınan toplamın başka bir alanda gösterilmesi. 
**Kullanılışı :** 
   XML dökümanında alt toplam alınacak sutuna TotalSummary="Sum" şeklinde tak eklenir. Eğer bu toplamın başka bir kontrolde gözükmesi isteniyor ise kontrole data source eklenir.
Datasource 'un SourceType tagına "Screen" RelatedProperty tagına gridin property'si , ControlValue tagına gösterilecek alanın propertysi yazılır. 
 
Örnek :
Gridde alt toplam alınması : 
```xml
<control FieldName="RequestDCol" ControlType="GridEdit">
<GridColumn FieldName = "UnitPrice" ControlType = "SpinEdit" Caption = "Birim Fiyat" TotalSummary="Sum"></GridColumn>
 </control>
```
Alt toplamın başka kontrolde gözükmesi
```xml
<control FieldName = "SumTotal"  ControlType = "TextEdit" Caption = "Toplam Fiyat">
<DataSource SourceType="Screen" RelatedProperty="RequestDCol" ControlValue="UnitPrice"></DataSource>
</control>
```
##	MAİNLİST’E FİLTER EKLEME NASIL YAPILIR?
Liste sayfalarımızda gerekli olan Filterları eklemek için uygulanacak adımlar aşağıda sıralanmıştır.
1.	Liste sayfamızda kullanılacak Filter XML’i  standart olarak çalıştığımız modulün klasorünün altına Filter klasorunun altına ObjeAdı+Filter.xml (Örnek:  INV/Filter/ WayBilFilter.xml) ismiyle eklenir.
2.	Oluşturulan xml dosyasının içersine arama kriterleri eklenir.
Ör:
```xml
<root Caption="İrsaliye" 
 MainObject="INV.ItemMCollection, INV, Version=1.0.0.0, Culture=neutral,  PublicKeyToken=null">
  <LayoutUpButtons ColumnCount="13">
    <section>
      <row>
        <cell>
          <control FieldName = "Operation_DocNo" 
             ControlType = "ComboEdit" Caption = "Banka Adı" Width="75">
            <DataSource  SourceType="Enum" Source="GNL.Operator, GNL, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null">
            </DataSource>
          </control>
        </cell>
        <cell>
          <control FieldName = "DocNo_1"
            ControlType = "TextEdit" CaptionVisible="false"  Width="100">
          </control>
        </cell>
        <cell>
          <control FieldName = "DocNo_2"
           ControlType = "TextEdit" CaptionVisible="false"  Width="100">
          </control>
        </cell>
        <cell>
          <control FieldName = "btnSearch" 
              ControlType = "Button" Caption = "ARA" Width="100">
            <ClientSideEvents Click="function(s, e) { 
                      myListPage.PerformCallback('L');
                }">
            </ClientSideEvents>
          </control>
        </cell>
      </row> 
</section>
  </LayoutUpButtons>
</root>
```
Yukarıda örneği verilen Filter Xml’inde tasarım tamamen kullanıcıya bırakılmıştır.**Ancak aşağıdaki adımlara dikkat edilmesi şarttır.**
-	Operation_ ismiyle başlayan ComboEdit mutlaka her satırda bulunmalıdır.Bu ComboEdit’in DataSource’ı GNL.Operator’den gelmesi
Şarttır.Ayrıca Fieldname verilirken mutlaka Operation_ isminden sonra ilişkili olacağı property’inin properyname’i verilmelidir.
Örnek(Operation_DocNo DocNo aramasında ilişkilendirilecek olan combo’nun adıdır.)
-	Her Arama kriteri ControlType’ı ne olursa olsun mutlaka iki ayrı control(Aralarında kriterinde kullanılmak üzre) olarak eklenmelidir.
Örnek DocNo_1  ve DocNo_2 şeklinde iki ayrı component olarak sisteme eklenmiştir.Dikkat edilcek husus propertyName’inden sonra 
İlki için _1 ikincisi için _2 eklerini FieldName’lere ilave etmektir.Tüm ekranlarda kullandığımız bütün kontrol tiplerini ve de 
Özelliklerini aynen kullanabilirsiniz.
-	Sadece bir tane eklemek üzre btnSearch butonu aynen (istenilen yere) eklenmelidir.
-	Tüm Arama kriterlerinde  aşağıdaki taglar default olarak(kendi objemize göre ) filter’a eklenmelidir.
```xml
<root Caption="İrsaliye" 
 MainObject="INV.ItemMCollection, INV, Version=1.0.0.0, Culture=neutral,  PublicKeyToken=null">
  <LayoutUpButtons ColumnCount="13">
    <section>
…….
              </section>
            </LayoutUpButtons>
```


3)APPD_FILTER tablosunda objemizin bulunduğu satırın FILTER_STRING kolonuna kullanacağımız FilterXml’in yolu verilir. (Ör:INV\Filter\WayBilFilter.xml)

##	MAİNLİST’E MASTER DETAY EKRAN NASIL YAPILIR?
APPD_COMMAND tablosunda Show komutun bulunduğu satırdaki ExtraParameters alanına aşağıda gibi değerler eklenir.
DetailObject: Detay objesi yazılır.
Relation : Master ile ilişkisi belirtilir.
Örnek :
MainList.aspx?MainObject='FIN.FinMCollection,FIN,Version=1.0.0.0,Culture=neutral,PublicKeyToken=null'&DetailObject='FIN.FinDCollection,FIN,Version=1.0.0.0,Culture=neutral,PublicKeyToken=null'&Relation=Id,FinMId&XMLName=Fin.xml&PageCode=FIN20031&DetailFilter=ForMaster=0

##	MAİNLİST’E İLK AÇILIŞTA KAYITLARIN OTOMATIK FILTRELİ GELMESİ NASIL YAPILIR?
APPD_COMMAND tablosunda Show komutun bulunduğu satırdaki ExtraParameters alanına aşağıda gibi değer eklenir.
DetailFilter: Filtre yapılacak property ve değeri yazılır.
Örnek :
MainList.aspx?MainObject='FIN.FinMCollection,FIN,Version=1.0.0.0,Culture=neutral,PublicKeyToken=null'&DetailObject='FIN.FinDCollection,FIN,Version=1.0.0.0,Culture=neutral,PublicKeyToken=null'&Relation=Id,FinMId&XMLName=Fin.xml&PageCode=FIN20031&DetailFilter=ForMaster=0

##	KART EKRANLARINA HTML KONTROL NASIL EKLENİR?
Kart ekranlarına html kontrollerin tümünü kullanabilirsiniz. (image,span,input,table,div ....) 
  Kullanılışı :
   XML dökümanında control tagının içerisine ControlType tipinine  "HtmlElement" caption tagınada Html elementleri yazılır.
Önemli Not :
html elementlerini yazarken küçüktür işareti yerine **&lt;** yazılması gerekmektedir.

<img src='images/homepageUyum.jpg' /> bunun yerine
&lt;img src='images/homepageUyum.jpg' /> bu yazılmalıdır.
Örnek :
  Ekrana resim yerleştirme :
```xml
<cell colspan="3" rowspan="3" >
  <control ControlType = "HtmlElement" Caption = "&lt;img src='images/homepageUyum.jpg' />"></control>
</cell>
```
Ekrana label yerleştirme :
```xml
<cell  colspan="3" rowspan="3" >
  <control ControlType = "HtmlElement" Caption = "&lt;span>Bu bir denemedir. &lt;/span>"></control>
</cell>
```
Ekrana table koyma
```xml
<cell  colspan="3" rowspan="3" >
  <control ControlType = "HtmlElement" Caption = "<table><tr><td>Deneme</td></tr></table>"></control>
</cell>
```

**Dikkat : Bu örnekte küçüktür işareti yerine *&lt;* yerleştirilmelidir. **Okunaklı olması açısından örnekte yerleştirilmemiştir.**  

##	KAYIT YAPARKEN SORU SORDURMA NASIL YAPABİLİRİM?
Kaydet butonuna bastıktan sonra , c# de kontrol yapıp kullanıcıya bir uyarı verdirmek istiyorsanız (Evet ve Hayır seçeneği ile)  bunu çok kolay bir şekilde yapabilirsiniz...
Örneğin : Fatura kaydet butonuna bastıktan sonra firmanın risk limitlerinin kontrol edip risk limitine yaklaşmıs ise , kullanıcıya "Bu firma risk limitine yaklaşmıştır" mesajı verip devam etmek istiyormusunuz diye soru sordurabilirsiniz. Evet cevabı verdiyse bu sefer "Tarih bugüne ait değil" deyip tekrar devam etmek istediğinize eminmisiniz diye soru sordurabilirsiniz.
Bu işlemi takibi için UyumObjectBase clasına PromptValue propert'isi eklenmiştir.
Kullanılışı : 
    Objenin insert, update  veya delete metodlarında uyarı fırlatmak için, numarası 10000 ile 11000 arasında UyumException fırlatmanız yeterli olacaktır. Uyarı fırlattıktan sonra kullanıcı eğer o uyarıya tamam dedi ise objede this.PromptValue yazarak gönderdiğiniz exception numarasını geri alabilirsiniz ve ona göre yeni uyarılar fırlatabilirsiniz...
Örnek Kullanım :
```cs
public override bool Insert(Uyum.Data.IDataComponent data)
{
      if (this.PromptValue == 0)
    {
        throw new UyumException(10000, new Exception("Tarih bugunün tarihinden küçük. Devam etmek istediğinizden eminmisiniz..."));
    }
    if (this.PromptValue == 10000)
    {
      throw new UyumException(10001, new Exception("Risk limiti doldu.Devam etmek istediğinizden eminmisiniz."));
    }
    return base.Insert(data);
}
```

##	EKRANDAN SERVER SIDE KODU NASIL ÇALIŞTIRABİLİRİM (CALLCSHARPCODE)?

Eğer c# kodu çağrılacak ise bir tane event tanımlamak gerekmektedir. Event’in eventID attribute’une bir tane id verilir. type attributune çağıracağınız metodun bağlı olduğu classın type bilgisi , methodName kısmına ise çağrılacak method adı yazılır. jsFunctionName attribute’une kod çalıştıktan sonra geri dönecek javascri,pt fonksiyonu belirtilir.
Burada çağırırken CallCSharpCode fonksiyonumuzu çağıracaksınız.Bu fonksiyon 3 tane parametre alır.Birincisi bizim size yolladımız ve geri beklediğimiz parametre(Key) , ikincisi işlem type ( 0 = menu item , 1 = button) ,
Üçüncüsü ise isterseniz c# tarafına göndermek istediğiniz parametreler.Bunları string olarak birleştirip ve birleştirikende aralarına ‘|’ ekleyerek ( tüm alt yapıda bu kullanılıyor ) bize yollayabilirsiniz. C# tarafında çağrılacak metoda bir sizden gelen parametre ,ikinci olarakta page’yi yolluyoruz.
Birincisi ( parametre) bir string : Key + ‘|’ + ID (ve varsa siz yolladıysanız) + ‘|’ + [sizden gelen prms].Bunu c# tarafında parse edip kullanabilirsiniz.

Javascript :
```js
function setCurRateTraGrid(result)
{
   SetControlValue('DocNo',result);
}
```
```xml
<ClientSideEvents ValueChanged="function(s, e) { 
                                 CallCSharpCode('AbtActGrpCodeVC',2,prm);   }">
</ClientSideEvents> 
``` 
gibi
```xml
<events>
    <event eventID="AbtActGrpCodeVC" Type="ZZ.WEB.FINGlobalClass, {DLL} "       MethodName="getCurRateTra" jsFunctionName="setCurRateTraGrid">
    </event>
 </events>
 ```
 ```cs
using Uyum.Objects;
using GNL;

public string Hesapla(string prm, BasePage page)
{
  int number = Convert.ToInt32(GlobalObj.gc.GetControlValue(page,"DocNo")); 
  ObjectLoadManifest manifest = new ObjectLoadManifest();
  manifest.Filter.Conditions=new SqlConditionManager();
  manifest.Filter.Conditions.Add(new SqlOperatorEquality("GNLD_COMPANY.CO_ID",number));
  manifest.SelectProperties.Add("CoCode");
  manifest.OrderBys.Add("CoCode");
  CompanyCollection companyCol = GlobalObj.Load<CompanyCollection>(manifest);
  return companyCol[0].CoCode;
}
```

## BUTTONEDIT ALANLARDA AŞAĞI OK TUŞUNA BASINCA HANGİ ALANLARIN GÖZÜKECEĞİ NASIL BELİRLENMEKTEDİR ?

Aşağı ok tuşuna bastığınızda gösterilecek alanlar şu kurallara göre çıkıyor.

1.  ReturnedProperties alanında belirtilen propertiler, appd_filter’da ListBrowsableString alanına bakılarak burda olanlar eklenir. (Standart yapı) 
2.  Object’de Description property var ise ve columnflags = none değil ise listenin sonuna eklenir.
3.  Object’de Description yok ise buttonedit alanın fieldname’i ListBrowsableString alanında aranır ve bundan sonra ilk bulduğu değer eklenir.
Örnek : 
```xml
<control FieldName="DocTraCode" ControlType="ButtonEdit" MaxLength="50" Width="150" Caption="Stok Hareket Kodu">
  <DataSource RelatedProperty="BranchCode"
              ReturnProperties="DocTraId;DocTraCode"
              ReturnedProperties="DocTraId;DocTraCode"
              SourceType="Command"
      Source="TempCoDocTraCollection.Show" ListPropertyTable="GNLD_DOC_TRA" ListPropertyName="DocTraCode" ProcessTypeMode="1">
  </DataSource>
</control>
```

ReturnedProperties ’ler DocTraId;DocTraCode
AppdFielter’da ListBrowsableString CoCode,DocTraCode,Description,Ispassive,SourceApp,InventoryStatus,IsDocDifferentCur,Ispassive olsun ekranda
DocTraCode ve Description alanı çıkar…

##	BİR KOŞULA GÖRE ZORUNLULUK NASIL YAPABİLİRİM?
Koşula göre değişecek kontrole validation yazarak bunu yapabilirsiniz. Validation içine javascript kodu yazarak e.isvalid = false; yazıp alert verirseniz zorunluluktan geçmez, e.isvalid = false yazarsanız zorunluluk kontrolünden geçer.
Örnek :
```xml
<ClientSideEvents Validation="function(s, e) {
isCostCenter = GetColumnValue('FinDCollection','IsCostCenterId');   
if(isCostCenter == 'True')
{
  alert(LangErr.FIN.CostCenterRequiredBecauseOfAccCode);
  e.isValid = false;
}
else { 
  e.isValid=true;
}
}">
</ClientSideEvents>
```
##	BİR BUTONA TIKLADIĞIMDA EKRANDAKİ ZORUNLULUK KONTROLLERİNİ CLENT SIDE TARAFINDA NASIL ÇAĞIRABİLİRİM?

Butonun altına aşağıdaki gibi yazarak yapabilirsiniz.
```xml
<ClientSideEvents Click="function(s, e) {
if(ASPxClientEdit.ValidateGroup('valid2',true)== false)
{
     alert('Girmeniz Gereken Zorunlu Alanlar Var');
     return false;
}
```

##	MAINLIST’IN LİSTELEME İŞLEMİNDE CUSTOM FILTER EKLEME NASIL YAPARIM?

Liste ekranına tanımlı bir filter xml dosyası var ise onun en altına tanımlı yok ise yeni xml dosyası tanımlanır ve bu xml dosyası appd_filter tablosunda filter_string alanında tanımlanır. Tanımlama aşağıdaki gibi yapılır.
```xml
  <PageLoadEvents>
    <AfterPageLoad Type="Senfoni.GNL.WEB.CompanyFilterCode,GNL.WEB,Version=1.0.0.0,Culture=neutral,PublicKeyToken=null" StrPrm=""  MethodName="AfterPageLoad" >
    </AfterPageLoad>
  </PageLoadEvents>
  ```

## MENÜ KLASÖR Ekleme
Klasör Ekleme Komut Tanım ekranından komut adını cmd prefixi ile başlatıp sabit bir komut yazmak gerekiyor.

## TIPS:
* Detay gridinde width ayarı yapabilmek adına her kolona ayrı ayrı ```xml  ServerAttribute="Width=200"  ``` attribute u eklenebilir. Bu sayede istenilen genişlik elde edilebilir.
