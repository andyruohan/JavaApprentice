������Ϸ��ɫ�й������ͷ��������ڴ�ս Boss ǰ���������״̬���������ͷ�������������ս Boss �󹥻����ͷ������½����ӱ���¼����ָ�����սǰ��״̬

###��ͳ�������
![img_1.png](��ͳ�������.png)

ȱ�㣺
1) һ������Ͷ�Ӧһ���������״̬�Ķ���������������Ϸ�Ķ���ܶ�ʱ�������ڹ�������Ҳ�ܴ�.
2) ��ͳ�ķ�ʽ�Ǽ򵥵������ݣ�new ������һ������������ٰ���Ҫ���ݵ����ݷŵ�����¶��󣬵���ͱ�¶�˶����ڲ���ϸ��

###����¼ģʽ
![img_2.png](����¼ģʽԭ��ͼ.png)
- Originator����Ҫ����״̬����
- Memento������¼�࣬���𱣴�ü�¼���� Originator �ڲ�״̬�� 
- Caretaker���ػ����࣬���𱣴�������¼���󣬿�ʹ�ü��Ϲ������Ч��  

�ŵ㣺
1) ���û��ṩ��һ�ֿ��Իָ�״̬�Ļ��ƣ�����ʹ�û��ܹ��ȽϷ���ػص�ĳ����ʷ��״̬
2) ʵ������Ϣ�ķ�װ��ʹ���û�����Ҫ����״̬�ı���ϸ��

###ʹ�ñ���¼ģʽ�����Ϸ��ɫ�ָ�ϵͳ
![img_3.png](ʹ�ñ���¼ģʽ�����Ϸ��ɫ�ָ�ϵͳ.png)
#####��Ҫ����״̬����
```java
@Getter
@Setter
public class GameRole {
    private int vit;
    private int def;

    //���� GameRole ״̬�� Memento ����
    public Memento createMemento() {
        return new Memento(vit, def);
    }

    //�� Memento ���󣬻ָ� GameRole ��״̬
    public void recoverGameRoleFromMemento(Memento memento) {
        this.vit = memento.getVit();
        this.def = memento.getDef();
    }

    //��ʾ��ǰ GameRole ��״̬
    public void display() {
        System.out.println("��Ϸ��ɫ��ǰ�Ĺ�������" + this.vit + " ��������" + this.def);
    }
}
```

#####����¼��
```java
@AllArgsConstructor
@Setter
@Getter
public class Memento {
    private int vit; //������
    private int def; //������
}
```

#####�ػ�����
```java
@Getter
@Setter
public class Caretaker {
    //�Ե��� GameRole ����һ��״̬
    private Memento memento;

    //�Ե��� GameRole ������״̬
    //private ArrayList<Memento> mementos;

    //�Զ�� GameRole ������״̬
    //private HashMap<String, ArrayList<Memento>> rolesMementos;
}
```

#####�ͻ��˷�����
```java
public class Client {

    public static void main(String[] args) {
        GameRole gameRole = new GameRole(); //������Ϸ��ɫ
        gameRole.setVit(100);
        gameRole.setDef(100);
        System.out.println("��boss��սǰ״̬");
        gameRole.display();

        Caretaker caretaker = new Caretaker();
        caretaker.setMemento(gameRole.createMemento()); //�ѵ�ǰ״̬����caretaker

        System.out.println("��boss��ս������");
        gameRole.setDef(30);
        gameRole.setVit(30);
        gameRole.display();

        System.out.println("��boss��ս��ʹ�ñ���¼����ָ���սǰ");
        gameRole.recoverGameRoleFromMemento(caretaker.getMemento());
        gameRole.display();
    }
}
```

###����¼ģʽ��ע�������ϸ��
 
1) �����ĳ�Ա�������࣬�Ʊػ�ռ�ñȽϴ����Դ������ÿһ�α��涼������һ�����ڴ�, �����Ҫע�� 
2) Ϊ�˽�Լ�ڴ棬����¼ģʽ���Ժ�ԭ��ģʽ���ʹ��

#####���õ�Ӧ�ó�����
1) ���ҩ��
2) ����Ϸʱ�Ĵ浵�� 
3) Windows ��� ctrl + z�� 
4) IE �еĺ��ˡ� 
5) ���ݿ���������
