![](https://googledrive.com/host/0B8v0ikF4z2BiR29YQmxfSlE1Sms/Progetti/Thrower/logo-64.png "Thrower Logo") Thrower
==================================================================================================================

*Fully managed library providing convenience methods to perform argument checks.*

## Summary ##

* Latest release version: `v3.0.1`
* Build status on [AppVeyor](https://ci.appveyor.com): [![Build status](https://ci.appveyor.com/api/projects/status/xjkp8gn0cf4s7qbg?svg=true)](https://ci.appveyor.com/project/pomma89/thrower)
* [Doxygen](http://www.stack.nl/~dimitri/doxygen/index.html) documentation: 
    + [HTML](https://goo.gl/iO6qZG)
    + [PDF](https://goo.gl/lZ7K9h)
* [NuGet](https://www.nuget.org) package(s):
    + [PommaLabs.Thrower](https://nuget.org/packages/Thrower/)

## Introduction ##

This library allows to write preconditions like the ones exposed in the following example:

```cs
/// <summary>
///   Simple example for Thrower.
/// </summary>
internal static class BankExample
{
    /// <summary>
    ///   My bank implementation.
    /// </summary>
    internal sealed class MyBank
    {
        /// <summary>
        ///   Stores whether this bank is open or not.
        /// </summary>
        private bool _isOpen;

        /// <summary>
        ///   The amount held into the bank.
        /// </summary>
        public decimal Amount { get; private set; }

        /// <summary>
        ///   Customers are very polite and say hello.
        /// </summary>
        /// <param name="helloMsg">The hello message.</param>
        /// <exception cref="ArgumentException">The hello message is null or blank.</exception>
        public void SayHello(string helloMsg)
        {
            // Preconditions
            Raise.ArgumentException.IfIsNullOrWhiteSpace(helloMsg, nameof(helloMsg), "Hello message is null or blank");

            Console.WriteLine(helloMsg);
        }

        /// <summary>
        ///   Deposits given amount into the bank.
        /// </summary>
        /// <param name="amount">A positive amount of money.</param>
        /// <exception cref="ArgumentOutOfRangeException">Amount is zero or negative.</exception>
        /// <exception cref="InvalidOperationException">Bank is closed.</exception>
        /// <exception cref="OverNineThousandException">Amount is over nine thousand!</exception>
        public void Deposit(decimal amount)
        {
            // Preconditions
            Raise.InvalidOperationException.IfNot(_isOpen, "Bank is still closed");
            Raise.ArgumentOutOfRangeException.IfIsLessOrEqual(amount, 0, nameof(amount), "Zero or negative amount");
            Raise<OverNineThousandException>.If(amount > 9000M, "You are too rich!");

            Amount += amount;
        }

        /// <summary>
        ///   Sends an email address from and to given addresses using the specified body.
        /// </summary>
        /// <param name="fromAddress">The address which sent the email. International characters are _not_ allowed.</param>
        /// <param name="toAddress">The address which will receive the email. International characters are allowed.</param>
        /// <param name="body">The message body.</param>
        /// <exception cref="ArgumentException">
        ///   Given email addresses are not valid. Given body is null, empty or blank.
        /// </exception>
        public void SendMail(string fromAddress, string toAddress, string body)
        {
            // Preconditions
            Raise.ArgumentException.IfIsNotValidEmailAddress(fromAddress, nameof(fromAddress), EmailAddressValidator.Options.AllowTopLevelDomains);
            Raise.ArgumentException.IfIsNotValidEmailAddress(toAddress, nameof(toAddress), EmailAddressValidator.Options.AllowInternational);
            Raise.ArgumentException.IfIsNullOrWhiteSpace(body, nameof(body), "The email body cannot be blank");

            Console.WriteLine($"From: {fromAddress}");
            Console.WriteLine($"To: {toAddress}");
            Console.WriteLine($"Message: {body}");
        }

        /// <summary>
        ///   Opens the bank.
        /// </summary>
        public void Open()
        {
            _isOpen = true;
        }
    }

    /// <summary>
    ///   Too rich for this bank.
    /// </summary>
    internal sealed class OverNineThousandException : Exception
    {
        /// <summary>
        ///   Without a custom message.
        /// </summary>
        public OverNineThousandException() : base("It's over nine thousand!")
        {
        }

        /// <summary>
        ///   With a custom message.
        /// </summary>
        /// <param name="msg">The custom message.</param>
        public OverNineThousandException(string msg) : base(msg + " - It's over nine thousand!")
        {
        }
    }

    /// <summary>
    ///   Simple example for Thrower.
    /// </summary>
    private static void Main()
    {
        var bank = new MyBank();

        try
        {
            // Say nothing!
            bank.SayHello("   ");
        }
        catch (ArgumentException ex)
        {
            // Polite people say meaningful things.
            Console.Error.WriteLine(ex.Message);
        }

        bank.SayHello("Good morning!"); // Everything OK!

        try
        {
            bank.Deposit(100);
        }
        catch (InvalidOperationException ex)
        {
            // Bank is still closed.
            Console.Error.WriteLine(ex.Message);
        }

        bank.Open();
        try
        {
            bank.Deposit(-1000);
        }
        catch (ArgumentOutOfRangeException ex)
        {
            // Cannot deposit a negative amount.
            Console.Error.WriteLine(ex.Message);
        }

        try
        {
            bank.Deposit(9001M);
        }
        catch (OverNineThousandException ex)
        {
            // Cannot deposit more than 9000.
            Console.Error.WriteLine(ex.Message);
        }

        bank.Deposit(10); // Everything OK!
        Console.WriteLine("Amount: " + bank.Amount);

        // Send an email with current amount.
        bank.SendMail("info@mybank.org", "юзер@екзампл.ком", $"Your current amount is {bank.Amount}");

        Console.Read();
    }
}
```

As of now, I do not have plans to expand Thrower beyond what it currently is. There are many ways in which it can be improved, I know, but as of now the library suits my needs and I have not much time to improve it. If I will have time, I will try to make it better, of course.

## Benchmarks ##

All benchmarks were implemented and run using the wonderful [BenchmarkDotNet](https://github.com/PerfDotNet/BenchmarkDotNet) library.

### Raise VS Throw ###

In this benchmark we try to understand how great is the speed difference between the .NET `throw` statement and our fluent syntax based on the `Raise` static class. As we can see by the results, the speed difference, if any, is really small.

```ini
Host Process Environment Information:

BenchmarkDotNet=v0.9.8.0
OS=Microsoft Windows NT 6.1.7601 Service Pack 1
Processor=Intel(R) Xeon(R) CPU X5650 2.67GHz, ProcessorCount=4
Frequency=10000000 ticks, Resolution=100.0000 ns, Timer=UNKNOWN
CLR=MS.NET 4.0.30319.42000, Arch=32-bit RELEASE
GC=Concurrent Workstation
JitModules=clrjit-v4.6.1590.0

Type=RaiseVsThrow  Mode=Throughput  GarbageCollection=Concurrent Workstation  

```
                                           Method |     Median |    StdDev | Gen 0 | Gen 1 | Gen 2 | Bytes Allocated/Op |
------------------------------------------------- |----------- |---------- |------ |------ |------ |------------------- |
                      Raise_ArgumentNullException | 16.9288 us | 0.5162 us |     - |     - |     - |              95,69 |
                RaiseStatic_ArgumentNullException | 21.9448 us | 0.4870 us |     - |     - |     - |             101,19 |
                      Throw_ArgumentNullException | 21.0086 us | 0.2489 us |     - |     - |     - |             101,19 |
       Raise_ArgumentOutOfRangeException_Integers | 16.1269 us | 0.2061 us |     - |     - |     - |              74,55 |
 RaiseStatic_ArgumentOutOfRangeException_Integers | 16.1793 us | 0.3492 us |     - |     - |     - |              72,83 |
       Throw_ArgumentOutOfRangeException_Integers | 16.0068 us | 0.2635 us |     - |     - |     - |              70,72 |
                      Raise_NotSupportedException | 48.4379 us | 2.4541 us | 71.00 |     - |     - |           2.518,03 |
                RaiseStatic_NotSupportedException | 47.8800 us | 4.5900 us | 73.82 |     - |     - |           2.583,65 |
                      Throw_NotSupportedException | 31.5260 us | 1.8334 us | 36.67 |     - |     - |           1.299,64 |

## About this repository and its maintainer ##

Everything done on this repository is freely offered on the terms of the project license. You are free to do everything you want with the code and its related files, as long as you respect the license and use common sense while doing it :-)
