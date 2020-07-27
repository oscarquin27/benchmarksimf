---
id: doc3
title: Validación de Negocio
---

Para la validación de negocio que se encontrará hard typed; se sugiere la utilización del tipo de variable **enum** para definir los codigos de los bancos y también
para los distintos tipos de moneda **Ccy**.

```csharp
[Flags]
        public enum CodBancos
        {
            Banco100 = 0156,
            BancoABN = 0196,
            Bancamiga = 0172,
            BancoActivo = 0171,
            BancoAgricola = 0166,
            BancoBicentenario = 0175,
            BancoCaroni =  0128,
            BancoDesarrollo = 0164,
            BancoVzla = 0102,
            BancoCaribe = 0114,
            BancoSoberano = 0149,
            BancoTesoro = 0163,
            BancoEspirito = 0176,
            BancoExterior = 0115,
            BancoIndustrial = 0003,
            BancoInternacionalDesarrollo = 0173,
            BancoMercantil = 0105,
            BancoNacionalCredito = 0191,
            BancoOccidental = 0116,
            BancoPlaza = 0138,
            BancoProvincial = 0108,
            BancoVenezolanoCredito = 0104,
            BancoBancrecer = 0168,
            BancoBanFANB = 0177,
            BancoBangente = 0146,
            BancoBanplus = 0174,
            BancoCitibank = 0190,
            BancoCorpBanca = 0121,
            BancoDelsur = 0157,
            BancoFondoComun = 0151,
            BancoInstitutoMunicipal = 0601,
            BancoMibanco = 0169,
            BancoSofitasa = 0137

        }
```
La razón para utilizar el tipo **Enum** se debe a que como tal él no reserva espacio de memoria ya que actúa como un **#define** entonces existe es en la etapa de compilación.
 
```csharp
static void Main(string[] args)
        {
            Console.WriteLine(sizeof(CodBancos)); // 4 Bytes en runtime memory
        }
```

Ahora la validación se puede hacer de la siguiente forma:

```csharp
[Benchmark]
        public (bool,bool,bool) ValidarBancoCcy()
        {
            bool success = false;
            bool success_2 = false;
            bool success_3 = false;
                
                CstmrCdtTrfInitnJson jsonFile = NetSystemDeserialization();
                success = Enum.IsDefined(typeof(CodBancos), int.Parse(jsonFile.CstmrCdtTrfInitn.PmtInf[0].DbtrAgt));
                success_2 = Enum.IsDefined(typeof(CodBancos), int.Parse(jsonFile.CstmrCdtTrfInitn.PmtInf[0].CdtrAgt));
                success_3 = Enum.IsDefined(typeof(Ccy), jsonFile.CstmrCdtTrfInitn.PmtInf[0].Amount.Ccy);
  
            return (success, success_2, success_3); //Cualquier valor que retorne false quiere decir que hay un error en el json
        }
```

Ahora el resultado del benchmark:

``` ini

BenchmarkDotNet=v0.12.1, OS=Windows 7 SP1 (6.1.7601.0)
Intel Core i7-3770 CPU 3.40GHz (Ivy Bridge), 1 CPU, 8 logical and 4 physical cores
Frequency=3420605 Hz, Resolution=292.3459 ns, Timer=TSC
.NET Core SDK=3.1.201
  [Host]     : .NET Core 3.1.4 (CoreCLR 4.700.20.20201, CoreFX 4.700.20.22101), X64 RyuJIT
  DefaultJob : .NET Core 3.1.4 (CoreCLR 4.700.20.20201, CoreFX 4.700.20.22101), X64 RyuJIT


```
|                   Method |     Mean |    Error |   StdDev |  Gen 0 | Gen 1 | Gen 2 | Allocated |
|------------------------- |---------:|---------:|---------:|-------:|------:|------:|----------:|
|          ValidarBancoCcy | 47.51 μs | 0.400 μs | 0.355 μs | 3.2959 |     - |     - |  13.54 KB |
| NetSystemDeserialization | 45.62 μs | 0.200 μs | 0.167 μs | 3.2959 |     - |     - |  13.48 KB |
|                IntParser | 45.75 μs | 0.088 μs | 0.082 μs | 3.2959 |     - |     - |  13.51 KB |

Se puede notar que la data asignada en el método ValidarBancoCcy puede ser proveniente de la Deserialización de NetSystem y posiblemente el int.Parse.

Una forma de demostrarlo es inyectando los valores manualmente sin tener que leer del archivo json. *Archivo json utilizado para las pruebas: INC.CDT.IMDT.001.01.JSON*

``` ini

BenchmarkDotNet=v0.12.1, OS=Windows 7 SP1 (6.1.7601.0)
Intel Core i7-3770 CPU 3.40GHz (Ivy Bridge), 1 CPU, 8 logical and 4 physical cores
Frequency=3420605 Hz, Resolution=292.3459 ns, Timer=TSC
.NET Core SDK=3.1.201
  [Host]     : .NET Core 3.1.4 (CoreCLR 4.700.20.20201, CoreFX 4.700.20.22101), X64 RyuJIT
  DefaultJob : .NET Core 3.1.4 (CoreCLR 4.700.20.20201, CoreFX 4.700.20.22101), X64 RyuJIT


```
|                   Method |        Mean |     Error |    StdDev |  Gen 0 | Gen 1 | Gen 2 | Allocated |
|------------------------- |------------:|----------:|----------:|-------:|------:|------:|----------:|
|          ValidarBancoCcy |    280.5 ns |   0.39 ns |   0.35 ns | 0.0114 |     - |     - |      48 B |
| NetSystemDeserialization | 45,234.1 ns | 216.40 ns | 168.95 ns | 3.2959 |     - |     - |   13808 B |
|                IntParser | 46,280.4 ns | 388.68 ns | 344.55 ns | 3.2959 |     - |     - |   13834 B |

En este caso, inyectando los valores manualmente, se puede observar que la asignación de memoria es casi nula, por lo que el trabajo realizado corresponde al deserializador.
Ahora realizando pruebas para 10 mil operaciones y utilizando el deserializador.

``` ini

BenchmarkDotNet=v0.12.1, OS=Windows 7 SP1 (6.1.7601.0)
Intel Core i7-3770 CPU 3.40GHz (Ivy Bridge), 1 CPU, 8 logical and 4 physical cores
Frequency=3420605 Hz, Resolution=292.3459 ns, Timer=TSC
.NET Core SDK=3.1.201
  [Host]     : .NET Core 3.1.4 (CoreCLR 4.700.20.20201, CoreFX 4.700.20.22101), X64 RyuJIT
  DefaultJob : .NET Core 3.1.4 (CoreCLR 4.700.20.20201, CoreFX 4.700.20.22101), X64 RyuJIT


```
|                   Method |          Mean |        Error |       StdDev |      Gen 0 |  Gen 1 | Gen 2 |    Allocated |
|------------------------- |--------------:|-------------:|-------------:|-----------:|-------:|------:|-------------:|
|          ValidarBancoCcy | 466,687.35 μs | 4,175.556 μs | 3,486.779 μs | 33000.0000 |      - |     - | 135365.34 KB |
| NetSystemDeserialization |      45.38 μs |     0.657 μs |     0.615 μs |     3.2959 |      - |     - |     13.48 KB |
|                IntParser |      46.28 μs |     0.910 μs |     0.710 μs |     3.2959 | 0.0610 |     - |     13.51 KB |

Se tomaría 0.046687 segundos realizar 10 mil validaciones de negocio, alojando 13 Mb de memoria.
