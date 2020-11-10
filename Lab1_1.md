using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace ConsoleApplication1
{
    public class CoffeeMachine
    {
        public enum State { Off, On, Error, Accept, Check, Bon_appetite };

        public class MenuItem
        {
            public string Name { get; set; }
            public int Price { get; set; }
            public int Number { get; set; }
            public int Milk { get; set; }

            public override string ToString()
            {
                return string.Format("{0}. {1}  Цена: {2} Кол-во молока в порции: {3} ml.", Number, Name, Price, Milk);
            }
        }

        State state = State.Off;
        List<MenuItem> menu = new List<MenuItem>();

        public event EventHandler<State> ChangeState;

        public CoffeeMachine(bool powerOn = false)
        {
            menu.Add(new MenuItem() { Number = 1, Name = "Чай", Price = 10, Milk = 0 });
            menu.Add(new MenuItem() { Number = 2, Name = "Эспрессо", Price = 12, Milk = 0 });
            menu.Add(new MenuItem() { Number = 3, Name = "Капучино", Price = 15, Milk = 50 });
            menu.Add(new MenuItem() { Number = 4, Name = "Ирландский виски", Price = 15, Milk = 25 });
            menu.Add(new MenuItem() { Number = 5, Name = "Шоколад", Price = 20, Milk = 0 });
            menu.Add(new MenuItem() { Number = 6, Name = "Латте", Price = 18, Milk = 50 });
        }

        public void Power(bool powerOn)
        {
            state = powerOn == true ? State.On : State.Off;
            Change();
        }

        private void Change()
        {
            if (ChangeState != null) ChangeState(this, state);
        }

        public string[] PrintMenu()
        {
            if (state != State.On) return new string[1] { "Автомат не готов." };
            return menu.Select(x => x.ToString()).ToArray();
        }

        public int Work(int menuNumber, int cash, int MilkVolume)
        {
            state = State.Check; Change();

            var menuItem = menu.Where(x => x.Number == menuNumber).FirstOrDefault();
            if (menuItem == null || cash - menuItem.Price < 0)
            {
                state = State.Error; Change();
                Console.WriteLine("Товар отсутствует в меню или у вас недостаточно денег.");
                return cash;
            }

            if (menuItem == null || MilkVolume - menuItem.Milk < 0)
            {
                state = State.Error; Change();
                Console.WriteLine("Недостаточно молока для приготовления напитка");
                return MilkVolume;
            }

            state = State.Accept; Change();

            state = State.Bon_appetite; Change();

            return cash - menuItem.Price;

        }
    }
        
    public class Operations
    {
        static void Main(string[] args)
        {
            Random rand = new Random();
            int cash = rand.Next(15, 25);

            int MilkVolume = 100;

            int Sugar = 1000;
            int a;
            string str;

            CoffeeMachine machine = new CoffeeMachine();
            machine.ChangeState += machine_ChangeState;
            machine.Power(true);

            Console.WriteLine("\t Меню \n");

            Console.WriteLine(string.Join("\n", machine.PrintMenu()));

            Console.Write("\n Объем молока в автомате {0} ml", MilkVolume);

            Console.WriteLine($"\nКол-во пакетиков сахара: {Sugar} штук");

            Console.Write("\nУ вас {0} грн.\nВыберите напиток:", cash);
          
            int select = int.Parse(Console.ReadLine());

            Console.WriteLine("Какое количество пакетиков сахара вы желаете?");

            str = Console.ReadLine();

            a = Convert.ToInt32(str);

            int s = Sugar - a;

            Console.WriteLine($"\nКол-во пакетиков сахара: {s} штук");

            Console.WriteLine("Готово. Остаток средств: {0} грн", machine.Work(select, cash, MilkVolume));


            Console.ReadKey(true);
        }

        static void machine_ChangeState(object sender, CoffeeMachine.State state)
        {
            switch (state)
            {
                case CoffeeMachine.State.On:
                    Console.WriteLine("Автомат включен.");
                    break;
                case CoffeeMachine.State.Error:
                    Console.WriteLine("Ошибка.");
                    break;
                case CoffeeMachine.State.Check:
                    Console.WriteLine("Идет приготовление напитка...");
                    break;
                case CoffeeMachine.State.Accept:
                    Console.WriteLine("Ваш напиток готов");
                    break;
                case CoffeeMachine.State.Bon_appetite:
                    Console.WriteLine("Приятного аппетита!!!");
                    break;
            }
        }
    }
}
