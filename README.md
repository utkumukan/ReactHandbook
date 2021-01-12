# React
Herkese merhaba. Bu çalışmada React ile ilgili birkaç konudan senaryolar üzerinde ilerleyerek bahsetmeye çalışıcam. Bütün örneklerin linklerini code bloğunun başlığına etiketledim. Paylaşılan code bloğunun başlığına tıklayarak ilgili linkten detaylı inceleyebilirsiniz.

## Nelerden bahsedicem?
 * bir componentin mount (first render) ve update (every re-render) durumları ve bağımlılıkları, 
 * agırlıklı olarak useEffect kullanımı, 
 * birkaç örnekle useRef kullanımı, 
 * child-parent ve parent-child iletişimleri ve
 * kendi hook'umuzu nasıl yazarız.

### Bir componentin mount (first render) ve update (every re-render) durumları ve bağımlılıkları

 * Bir componentin mount edilmesi durumu hepimizin de bildiği gibi ilk kez çağırılması durumudur. Eğer basit bir componentimiz varsa, mount edilme durumunda çok birşey yapmamıza gerek yoktur. Fakat kompleks bir componente sahipsek, mesela API çağrıları yapan, bu durumda useEffect, useRef gibi hook'lara ihtiyaç duyarız. Bu konuyu ilerleyen senaryolarda useEffect'le birlikte işliycem.

 * Update durumu ise bildiğimiz gibi componentimizin tekrar render edilmesi durumudur. **Bu durumun gerçekleşmesi için ya state'imiz güncellenmiş olmalı ya da props'umuz.** Bu konu da aslında incelenmesi gereken farklı açılara sahip olduğu için ilerleyen senaryolarda işlenecek fakat basit bir örnek üzerinden temel haline bakalım.

_[React-Ornek-1](https://codesandbox.io/s/react-ornek-1-mb5bx)_
 ```javascript
    const Child = (props) => {
        const { year } = props;

        return (
            <div>
                <h2>{year}</h2>
            </div>
        );
    };

    const App = () => {
        const [year, setYear] = useState("2020");

        const handleClick = () => {
            setYear("2021");
        };

        return (
            <div className="App" onClick={handleClick}>
                <h1>Gelecek Varlık</h1>
                <Child year={year} />
                <button onClick={handleClick}>Yeni Yılı Getir</button>
            </div>
        );
    };
 ```

Örnekte de görüldüğü üzere App componentinin state durumunu yönettiği ve bu state'i Child component'i ile paylaştığı bu senaryomuzda butona tıklanması sonucu year state'i güncelleniyor. **State** güncellendiği için önce App komponent kendini re-render ediyor, App componentinin state'i Child componentinin **props**'u olduğu için haliyle Child componenti de props değşikliğinden dolayı re-render oluyor.