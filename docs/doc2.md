---
id: doc2
title: NewtonSoft vs System vs Utf8 Json Deserializer
---

## Json Deserialization

El proceso se corrió usando tasks para cada uno de los distintos serializadores con 10 mil iteraciones cada uno. 

```ini
BenchmarkDotNet=v0.12.1, OS=Windows 7 SP1 (6.1.7601.0)
Intel Core i7-3770 CPU 3.40GHz (Ivy Bridge), 1 CPU, 8 logical and 4 physical cores
Frequency=3420605 Hz, Resolution=292.3459 ns, Timer=TSC
.NET Core SDK=3.1.201
  [Host]     : .NET Core 3.1.4 (CoreCLR 4.700.20.20201, CoreFX 4.700.20.22101), X64 RyuJIT
  DefaultJob : .NET Core 3.1.4 (CoreCLR 4.700.20.20201, CoreFX 4.700.20.22101), X64 RyuJIT

```
|                   Method  |        Mean |	    Error |	    StdDev |	    Gen 0	|       Gen 1	|       Gen 2 |	    Allocated |
|---------------------------|-------------|-----------|------------|----------------|---------------|-------------|---------------|
| NewTonSoftDeserialization |	213.5 ms  |	18.27 ms  |	53.58 ms   |	50833.3333	|   13500.0000  |	3500.0000 |	    185.25 MB |
|       UTF8Deserialization |	199.7 ms  |	29.59 ms  |	87.24 ms   |	52000.0000  |	13500.0000  |	3250.0000 |	    200.4 MB  |
|  NetSystemDeserialization	|   203.5 ms  |	21.39 ms  |	63.06 ms   |	33500.0000  |	8750.0000   |	2000.0000 |	    124.36 MB |


Net System Deserialization tiene notablemente menos memoria alocada para la misma cantidad de procesos y la diferencia de tiempo de ejecución es relativamente
parecida a la de UTF8 Deserialization.

El codigo utilizado para generar el benchmark es el siguiente:

```csharp
        [Benchmark]
         public void NetSystemDeserialization()
        {
            Parallel.For(0, 10000, async i => { //Se utilizo Parallel para ver de que forma concurrente
             await Task.Run(() => 
            {
                string jsonString = File.ReadAllText(fileName);
                CstmrCdtTrfInitnJson serializadorSystem = System.Text.Json.JsonSerializer.Deserialize<CstmrCdtTrfInitnJson>(jsonString);
                return serializadorSystem;
                });
            });
        }
```