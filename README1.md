# Implementación de un Procesador Pipeline de 5 Etapas en Verilog

---

## 💻 Simulación de Ondas

Comprobación del funcionamiento del hardware segmentado y la validación de las etapas de ejecución a través de ModelSim:

![Simulación de Ondas](wavespipeline.jpeg)

## 🗺️ Datapath de la Arquitectura

El siguiente diagrama representa la ruta de datos completa implementada en el hardware, mostrando las 5 etapas del pipeline y los lazos de adelantamiento (Forwarding) controlados por la Hazard Unit:

```mermaid
graph LR
    %% Definición de Estilos (Con texto en negro)
    classDef fetch fill:#e1f5fe,stroke:#0288d1,stroke-width:2px,color:#000000;
    classDef decode fill:#efebe9,stroke:#5d4037,stroke-width:2px,color:#000000;
    classDef execute fill:#e8f5e9,stroke:#388e3c,stroke-width:2px,color:#000000;
    classDef memory fill:#fff3e0,stroke:#f57c00,stroke-width:2px,color:#000000;
    classDef wb fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#000000;
    classDef hazard fill:#ffebee,stroke:#c62828,stroke-width:2px,color:#000000;

    %% Bloques Principales por Etapa
    subgraph Etapa_IF [IF - Fetch]
        PC[Program Counter]
        IM[Memoria de Instrucciones]
        Add4[Sumador PC + 4]
    end

    subgraph Etapa_ID [ID - Decode]
        RF[Banco de Registros]
        CU[Unidad de Control]
        SignExt[Extensión de Signo]
    end

    subgraph Etapa_EX [EX - Execute]
        ALU[ALU]
        MuxA[Mux Forwarding A]
        MuxB[Mux Forwarding B]
    end

    subgraph Etapa_MEM [MEM - Memory]
        DM[Memoria de Datos]
    end

    subgraph Etapa_WB [WB - WriteBack]
        MuxWB[Mux WriteBack]
    end

    subgraph Control_Riesgos [Control de Riesgos]
        HU[Hazard Unit]
    end

    %% Conexiones de la Ruta de Datos
    PC -->|Dirección| IM
    PC -->|PC_out| Add4
    Add4 -->|PC + 4| PC

    IM -->|Instrucción| CU
    IM -->|Rs, Rt, Rd| RF
    IM -->|Inmediato bruto| SignExt

    RF -->|RD1| MuxA
    RF -->|RD2| MuxB
    SignExt -->|Inmediato Ext| MuxB
    
    MuxA -->|Operando A| ALU
    MuxB -->|Operando B| ALU

    ALU -->|Resultado ALU| DM
    ALU -->|Resultado ALU| MuxWB

    DM -->|Dato Leído| MuxWB
    MuxWB -->|Dato a Escribir / WD3| RF

    %% Conexiones Hazard Unit y Forwarding (Lazos de retorno)
    HU -.->|Stall / Flush| PC
    HU -.->|Stall / Flush| CU
    
    ALU -.->|Forwarding MEM a EX| MuxA
    ALU -.->|Forwarding MEM a EX| MuxB
    
    MuxWB -.->|Forwarding WB a EX| MuxA
    MuxWB -.->|Forwarding WB a EX| MuxB
    
    %% Asignación de Estilos a Clases
    class PC,IM,Add4 fetch;
    class CU,RF,SignExt decode;
    class MuxA,MuxB,ALU execute;
    class DM memory;
    class MuxWB wb;
    class HU hazard;
