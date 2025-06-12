# The TACO specification

The terms "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document follow the definitions from [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt). The term "data user" refers to any individual or system accessing the data, "data provider" refers to the entity responsible for offering the data.

## Version and schema

**This is version 0.2.0 of the TACO specification**. The core data structures are defined through [UML diagrams](https://github.com/tacofoundation/specification/tree/main/diagrams) for easy inspection. To validate TACO-compliant files, use the [TACO Toolbox](https://github.com/tacofoundation/taco-toolbox), which is available in C++, Python, R, and Julia. **Future versions must remain backward-compatible with this one.**

### Summary

Geospatial communities have long relied on standardised formats for raster and vector data, enabling interoperability, stable tool development, and long-term data preservation. In contrast, Artificial Intelligence (AI)-ready datasets, particularly those derived from Earth Observation (EO), lack equivalent conventions. As a result, data producers often adopt ad hoc file structures, loosely defined formats, and inconsistent semantic encodings. This fragmentation hinders interoperability, complicates reuse, and undermines reproducibility. We argue that the lack of a standard format represents a structural bottleneck to scalable scientific progress, especially in the era of foundation models, where diverse datasets must be combined for effective training and performance evaluation in downstream tasks. To address this, we introduce TACO: a comprehensive specification that defines a formal data model, a cloud-optimized on-disk layout, and an API for creating and accessing AI-ready EO datasets.

### Introduction

The rapid increase in Earth Observation (EO) data, combined with advances in AI and cloud computing, has unlocked new opportunities for scientific discovery and operational monitoring (*Montillet et al., 2024*; *Eyring et al., 2024; *Hagos et al., 2022*). Modern applications range from methane superemitter detection (*Vaughan et al., 2024*) and burned area estimation (*Ribeiro et al., 2023*) to biodiversity tracking (*Yeh et al., 2021*) and global-scale weather forecasting (*Rasp et al., 2020; Bi et al., 2023*). These efforts increasingly rely on data-driven models, which require large volumes of curated, structured, and accessible EO data (*Reichstein et al., 2019*). However, preparing AI-ready EO datasets continues to be a significant challenge (*Sambasivan et al., 2021*; *Francis & Czerkawski, 2024*). Most datasets require extensive preprocessing and reformatting before being integrated into AI pipelines, and only a small fraction are usable `out of the box`. Although AI-ready EO datasets have grown substantially, with more than 500 now cataloged (*Schmitt et al., 2023*), they still lack a unified structure and consistent metadata conventions. This fragmentation hinders reproducibility, limits interoperability, and slows the development of AI (*Dimitrovski et al., 2023*; *Long et al., 2021*). These issues are especially critical for training foundation models, which rely on combining diverse sources (*Marsocci et al., 2024*).

Insights from scientific communities can guide the development of standardised, AI-ready EO datasets. Fields such as climate science and geographic information systems (GIS) have long struggled with data standardisation and provide valuable lessons through widely adopted formats like NetCDF (_Treinish & Gough, 1987_; _Rew & Davis, 1990_) and GeoTIFF (_Ritter & Ruth, 1997_; _Devys et al., 2019_). NetCDF was initially created as a binary format for multi-dimensional scientific data, with applications in atmospheric and climate sciences, but not limited to it. With broader use in climate science, it became evident that the standard NetCDF specification required complementary conventions to adequately represent the metadata needs of the domain. This realization led to the development of several metadata conventions, most notably the CF (Climate and Forecast) Conventions (*Eaton et al., 2024*), aimed at standardising the description of scientific variables, coordinates, and attributes. Although these conventions significantly improved interoperability, their __`text-based definitions`__ introduced ambiguities and made consistent implementation difficult. To address this, formal data models, such as the CF data model (*Hassell et al., 2017*), were introduced years later, offering a structured and unambiguous interpretation of what CF-compliant data means. GeoTIFF, in contrast, took a more pragmatic approach. Designed to facilitate the exchange of raster data between GIS applications (*Ritter & Ruth, 1997*), GeoTIFF embeds minimal but critical metadata, specifically the coordinate reference system (CRS) and geotransform, directly within the file (*Devys et al., 2019*). GeoTIFF, unlike NetCDF, was not developed with a comprehensive semantic model in mind. However, its simplicity and user-friendly design have led to widespread adoption.

In hindsight, both cases underscore the importance of maintainability. Crucially, both NetCDF and GeoTIFF have survived because active communities emerged around them, building tools and practices that reinforced and extended the specifications over time (*Devys et al., 2019*; *Maso et al., 2023*; *Eaton et al., 2024*). For CF-compliant NetCDF datasets, the experience highlighted the limitations of relying only on text-based definitions: as the authors of the CF data model argue in their conclusion, `creating an explicit data model before the CF conventions were written would arguably have been preferable. A data model encourages coherent implementations, which could be file storage syntaxes or software codes` (_Hassell et al., 2017_). In contrast, GeoTIFF illustrates how a well-defined minimal standard focused on a specific use case can achieve broad interoperability without necessitating a complex data model. These lessons highlight the need to balance formal rigor with practical simplicity. Given the inherent complexity of AI-ready EO datasets, we strongly support the development of a formal data model; however, whenever possible, it should be designed around the tools and workflows practitioners use on a daily basis to facilitate smooth adoption.

The FAIR principles (*Wilkinson et al., 2016*), Findability, Accessibility, Interoperability, and Reusability, provide a useful framework to address the challenges faced by the AI-ready EO datasets. Regarding **Findability**, web standardised metadata schemas (i.e., Schema.org, _Guha et al. 2016_) are rarely used to describe AI-ready EO datasets, limiting their visibility in search engines and data catalogs (_Benjelloun et al., 2024_). In terms of **Accessibility**, data access often depends on manual downloads or custom APIs rather than scalable, cloud-native formats that support partial or selective retrieval. With respect to **Interoperability**, the wide variety of formats, with differing conventions for byte layout, chunking strategies, compression, and explicit metadata, creates barriers to seamless integration across datasets. Finally, on **Reusability**, many datasets lack clear licenses, provenance, or documentation, making them difficult to audit, cite, or extend.

To close these gaps, we propose TACO (Transparent Access to Cloud Optimized Datasets), a FAIR-compliant, cloud-optimized specification for organizing AI-ready EO datasets. TACO files are self-contained, portable, and complete, encapsulating all the information required for sample interpretation without relying on external files or software dependencies. Built on widely supported technologies like GDAL and Apache Parquet, TACO allows for seamless integration across multiple programming languages.

<p>&nbsp;</p>
<a name="fig1">
<p align="center">
  <img src="https://github.com/user-attachments/assets/17a84407-0cd1-4da4-9e0d-2e22beda7087" width="50%">
</p>
</a>
<sub><strong>Figure 1:</strong> Conceptual organization of the TACO Specification. The Data Model (A) is composed of two layers: Logical Structure (describing the relationships between data and metadata) and Semantic Description (standardised metadata definitions). These layers collectively define the Data Format (B), specifying how data is stored, which can be created and accessed through a dedicated API (C) consisting of the ToolBox (for creation) and the Reader (for reading).</sub>
<p>&nbsp;</p>

### The specification

The TACO specification defines the data model, file format, and API ([**Figure 1**](#fig1)). Here, the **_data model_** refers to an abstract representation of a dataset that defines the rules, constraints, and relationships connecting metadata to the associated data assets ([**Figure 2**](#fig2)). The **_data format_** defines the physical representation of the dataset, specifying how data and metadata are encoded, stored, and organized. Finally, the API specifies the programmatic methods and conventions by which users and applications can interact with TACO-compliant datasets. By providing a unique and well-structured interface, the API abstracts the underlying complexity of the data format and data model, allowing data users to query, modify, and even integrate multiple TACO datasets.

#### The Data Model

The logical structure of the TACO data model is illustrated in the UML diagram in [**Figure 2**](#fig2). At its core, a TACO dataset is defined as a structured collection of minimal self-contained data units, called SAMPLEs, organized within a container, called TORTILLA, and enriched by dataset-level metadata.

<p>&nbsp;</p>
<a name="fig2">
<p align="center">
<img src="https://github.com/user-attachments/assets/f41109aa-357f-4a2c-b348-e39192a9ccc6" alt="TACO logical structure" width="75%">
</p>
</a>
<sub><strong>Figure 2:</strong> TACO logical structure. A <code>SAMPLE</code> encapsulates raw data and metadata, with a pointer to a <code>DataSource</code>. Supported data sources include <code>GDALDataset</code>, <code>BYTES</code>, and <code>TORTILLA</code>. TACO extends <code>TORTILLA</code> by adding high-level dataset metadata.</sub>
<p>&nbsp;</p>


A SAMPLE represents the minimal self-contained and smallest indivisible unit for AI training and evaluation. Each SAMPLE encapsulates the actual data and metadata ([**Figure 3**](#fig3)). Importantly, each SAMPLE contains a pointer to a DataSource that specifies how to access the underlying data. A SAMPLE supports three primary DataSource types: (i) GDALDataset, for raster or vector data readable by the GDAL library; (ii) BYTES, representing raw byte streams for unsupported or custom formats; and (iii) TORTILLA. While the BYTES option is available, GDALDataset is recommended for partial read support.

<p>&nbsp;</p>
<a name="fig3"></a>
<p align="center">
  <img src="https://github.com/user-attachments/assets/52dad4b8-d680-4f43-b666-23572e48df2e" alt="Semantic description of SAMPLE metadata" width="80%">
</p>
<sub><strong>Figure 3:</strong> Semantic description of the <code>SAMPLE</code> metadata. The <code>Metadata</code> class contains essential file identification and storage fields. An abstract <code>Extension</code> class defines the interface for optional metadata, allowing for expansion. Specific extensions (marked with <code>&lt;&lt;Extension&gt;&gt;</code> in the header) like <code>STAC</code>, <code>RAI</code>, <code>STATS</code>, <code>Flood</code>, and <code>Methane</code> inherit from <code>Extension</code>, each adding domain-specific attributes. This design enables adding extensions without modifying the core <code>Metadata</code> structure.</sub>
<p>&nbsp;</p>

The TORTILLA serves as a container that manages multiple SAMPLE instances. All SAMPLEs within a TORTILLA share a uniform metadata schema, enabling the combined metadata to be represented as a dataframe. Since TORTILLA implements the DataSource interface, it can be referenced within a SAMPLE, enabling recursive nesting of TORTILLA containers. This design supports the representation of hierarchical datasets while preserving the modularity and self-contained nature of individual SAMPLEs. Building upon TORTILLA, the TACO class extends this container structure by adding comprehensive dataset-level metadata ([**Figure 4**](#fig4)). This additional metadata provides a semantic collection overview, supporting dataset management, discovery, and interoperability.

<p>&nbsp;</p>
<a name="fig4"></a>
<p align="center">
  <img src="https://github.com/user-attachments/assets/e522fc11-7cc1-4670-a836-3491c1c2b1c2" alt="Semantic description of SAMPLE metadata" width="80%">
</p>
<sub><strong>Figure 4:</strong> Semantic description of the TACO dataset-level metadata. Core dataset information is structured in the Metadata class, linking mandatory and optional fields. Extensions, modeled through the abstract Extension class, allow modular inclusion of additional metadata such as RAI, Publications, and Sensor information, ensuring flexibility and scalability.</sub>
<p>&nbsp;</p>

#### Semantic Description

This section defines the structure of the metadata associated with each individual SAMPLE ([**Figure 3**](#fig3)) and with the TACO dataset ([**Figure 4**](#fig4)) as a whole. Metadata is organized into three categories: (1) Core (required fields), (2) Optional (non-essential fields providing additional context or supporting specific functionalities), and (3) Automatic (fields automatically generated by the TACO API; generation is based exclusively on core metadata and never on optional fields).

<p>&nbsp;</p>
<a name="tab1"></a>
<table>
  <thead>
    <tr>
      <th><strong>Field</strong></th>
      <th><strong>Type</strong></th>
      <th><strong>Details</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>tortilla:id</code></td>
      <td>String</td>
      <td><strong>CORE</strong>. Unique identifier for each item.</td>
    </tr>
    <tr>
      <td><code>tortilla:file_format</code></td>
      <td>String</td>
      <td>
        <strong>CORE</strong>. The format name <strong>MUST</strong> follow the GDAL naming convention. For example:
        <ul>
          <li>GeoTIFF files use the format name <code>GTiff</code>.</li>
          <li>JPEG files use the format name <code>JPEG</code>.</li>
        </ul>
        <strong>Additional Supported Formats:</strong>
        <ul>
          <li><code>BYTES</code>: Used for data formats not supported by GDAL.</li>
          <li><code>TORTILLA</code>: Used when the file represents a nested TORTILLA structure.</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td><code>tortilla:offset</code></td>
      <td>Long</td>
      <td><strong>AUTOMATIC</strong>. Byte offset where the item’s data begins in the file. This field is automatically generated by the <code>taco-toolbox</code>.</td>
    </tr>
    <tr>
      <td><code>tortilla:length</code></td>
      <td>Long</td>
      <td><strong>AUTOMATIC</strong>. Number of bytes that the item’s data occupies. This field is automatically generated by the <code>taco-toolbox</code>.</td>
    </tr>
    <tr>
      <td><code>tortilla:data_split</code></td>
      <td>String</td>
      <td>
        <strong>OPTIONAL</strong>. The data split type. <strong>MUST</strong> be one of the following:
        <ul>
          <li><code>train</code>: Training data.</li>
          <li><code>test</code>: Testing data.</li>
          <li><code>validation</code>: Validation data.</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>
<p><strong>Table 1:</strong> Core Schema for <code>SAMPLE</code> Metadata</p>
<p>&nbsp;</p>

At the `SAMPLE` level, two core attributes are required: `tortilla:id`, a unique string that identifies each `SAMPLE`, and `tortilla:file_format`, which specifies the data format—either `TORTILLA`, `BYTES`, or any format supported by GDAL. An optional field, `tortilla:data_split`, indicates the dataset partition to which the sample belongs (e.g., training, validation, or testing). Additionally, the fields `tortilla:offset` (denoting the position within a TORTILLA archive) and `tortilla:length` (the sample's size) are automatically computed by the TACO API ([**Table 1**](#tab1)). The current specification supports three optional extensions: STAC, Responsible AI (RAI), and sample statistics (STATS), which are described in detail in the [`SAMPLE` Extensions section](#sample-level-extension).

At the dataset level, TACO defines a `Metadata` class that encapsulates both core and optional fields describing the dataset’s provenance, structure, and content ([**Table 2**](#tab2)). Core fields include a persistent identifier (`id`), versioning information (`taco_version`, `dataset_version`), spatiotemporal coverage (`extent`), a human-readable description (`description`), licensing details (`licenses`), and contact information for both dataset providers (`providers`) and the individual responsible for converting the data into TACO (`data_curator`). Several of these core fields employ nested structures or lists to represent complex information. For example, both `providers` and `data_curator` are modeled as lists of `Contact` objects, each containing attributes such as name, affiliation, and email. The `extent` field uses nested list structures to capture spatial and temporal bounds, while the `licenses` field is represented by a `Licenses` class that can wrap one or more license entries.

Optional fields in the `Metadata` class include a dataset title, descriptive keywords, and high-level information about intended use, such as the task type and split strategy. Links to external resources can be provided via the optional `raw_link` and `discuss_link` fields, both represented by a `Hyperlink` class that includes an `href` and a textual `description`. TACO metadata is designed to be extensible: additional modules can be integrated by inheriting from an abstract `Extension` class. Check the [`TACO` Extensions section](#taco-level-extension) for more details.

<p>&nbsp;</p>

| **Field**         | **Type**                                           | **Details**                                                                                                           |
| ----------------- | -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `id`              | String                                             | **CORE**. A unique identifier for the dataset.                                                                        |
| `taco_version`    | String                                             | **CORE**. The version of the TACO specification.                                                                      |
| `dataset_version` | String                                             | **CORE**. Version of the dataset.                                                                                     |
| `description`     | String                                             | **CORE**. Description of the dataset.                                                                                 |
| `licenses`        | List of strings                                    | **CORE**. License(s) of the dataset. It is recommended to use [SPDX License identifiers](https://spdx.org/licenses/). |
| `extent`          | [Extent Object](#extent-object)                    | **CORE**. Spatial and temporal extents.                                                                               |
| `providers`       | List of [Person Objects](#person-object)           | **CORE**. A list of persons who participated in the creation of the dataset.                                          |
| `curators`        | List of [Person Objects](#person-object)           | **CORE**. A list of persons responsible for converting the dataset to TACO compliance.                                |
| `title`           | String                                             | **OPTIONAL**. Title of the dataset. Maximum length: 250 characters.                                                   |
| `keywords`        | List of strings                                    | **OPTIONAL**. List of keywords describing the dataset.                                                                |
| `task`            | [Task Object](#task-object)                        | **OPTIONAL**. Refers to the most relevant task defined by the TACO specification.                                     |
| `split_strategy`  | [Split Strategy Object](#split-strategy-extension) | **OPTIONAL**. Chosen from an explicit list of method names.                                                           |
| `discuss_link`    | [HyperLink Object](#hyperlink-object)              | **OPTIONAL**. A link to a discussion forum or community page.                                                         |
| `raw_link`        | [HyperLink Object](#hyperlink-object)              | **OPTIONAL**. Link to the raw dataset (if not in native TACO format).                                                 |

<p>&nbsp;</p>


#### Data format

The **TORTILLA** and **TACO** file formats are designed to efficiently store large-scale datasets using a binary serialization scheme ([**Figure 5**](#fig5)). Each TORTILLA file enforces a consistent schema and metadata structure across all its samples. Metadata is stored in the **FOOTER** using Apache Parquet, while the corresponding sample data is stored as a Binary Large Object (**BLOB**). Each row in the Apache Parquet file corresponds to a distinct `SAMPLE` object. The **BLOB** and the **FOOTER** are combined into a single file, constituting the TORTILLA format (see [**Figure 5**](#fig5)). Notably, the format enables partial reads of the **BLOB** during sample-level access, while the **FOOTER** is read entirely only once at load time.

<p>&nbsp;</p>
<a name="fig5"></a>
<p align="center">
  <img src="https://github.com/user-attachments/assets/47cf46be-6f40-4aec-bf97-9a674021bcfa" alt="Semantic description of SAMPLE metadata" width="80%">
</p>
<sub><strong>Figure 5:</strong> Structure of the TACO and TORTILLA file format, used as the underlying container for SAMPLEs. The format consists of a 200-byte static header followed by a dynamic segment. The static section encodes file-level metadata including a magic number (MB), footer offset (FO) and length (FL), data partition (DP), and pointers to the metadata collection (CO and CL, only for TACO). The dynamic section serializes data blobs (DATA), sample-level descriptors (FOOTER), and, in the case of TACO files only, a dataset-level metadata block (COLLECTION) encoded in UTF-8 JSON.</sub>
<p>&nbsp;</p>

A **TACO** file extends the TORTILLA format by appending dataset-level metadata (the **COLLECTION**), encoded in JSON at the end of the file. This design ensures that both TORTILLA and TACO files are self-contained, portable, and complete, encapsulating all information required to interpret samples without reliance on external files or software dependencies.

Each file begins with a fixed 200-byte **HEADER** that includes a 2-byte magic number, an 8-byte offset and length for the **FOOTER**, and an 8-byte data partition count indicating the dataset's number of segments. This count allows the TACO API to verify dataset completeness and reconstruct the full archive correctly. TACO files introduce two additional 8-byte fields for the **COLLECTION** offset and length. Both formats reserve unused space in the header for future use: 174 bytes in TORTILLA and 158 bytes in TACO.

The TACO API (Section [**API**](#api)) automatically generates certain fields based on the input data. For instance, it records sample-level offsets and lengths as columns in the **FOOTER**, enabling efficient random access to individual samples (illustrated by the red dotted line in [**Figure 5**](#fig5)). To support multi-language interoperability and partial reads, TACO relies on GDAL’s Virtual File System (VFS), particularly the `/vsisubfile/` handler, which allows byte ranges within a TACO file to be treated as standalone `GDALDataset` objects. This enables fast random access without reading the entire **BLOB** region. TACO also supports cloud-optimized access, leveraging additional GDAL VFS handlers such as `/vsicurl/`, `/vsis3/`, `/vsiaz/`, `/vsigs/`, `/vsioss/`, and `/vsiswift/`, ensuring high-performance reads across diverse cloud storage platforms.

<p>&nbsp;</p>
<a name="fig6"></a>
<p align="center">
  <img src="https://github.com/user-attachments/assets/ac910f8e-84d8-4b1a-bcbe-d46dba349692" width="80%">
</p>
<sub><strong>Figure 6:</strong> This diagram illustrates the key components of the TACO Toolbox API and their relationships. The Toolbox is responsible for creating, editing, and mapping between standards.</sub>
<p>&nbsp;</p>

#### API

The TACO API consists of two main components: the **Toolbox** ([**Figure 6**](#fig6)) and the **Reader** ([**Figure 7**](#fig7)). The Toolbox provides data classes for the core TACO models—`SAMPLE`, `TORTILLA`, and `TACO`—enabling users to define and modify dataset structures entirely through code. It includes a `create()` method that serializes both data and metadata into fully compliant TACO or TORTILLA files. Additionally, the `edit()` method allows users to update existing files, whether adjusting the `COLLECTION` or the `FOOTER`.

Format conversion is supported through optional utilities such as `tortilla2taco()`, `taco2tortilla()`, `footer2geoparquet()`, and `footer2geoparquetstac()`. Exporters like `collection2stac()`, `collection2croissant()`, `collection2datacite()`, and `collection2datacard()` enable collection-level metadata generation in STAC, Croissant, DataCite, or Markdown formats.

The **Reader** component provides a simple interface to load and interact with TACO and TORTILLA files. It implements a `load()` function that retrieves the `FOOTER` and, if called with `collection=True`, also returns the `COLLECTION`. A `compile()` function must also be provided to create smaller subsets of existing TACO or TORTILLA files.

The Reader is designed to operate within a DataFrame interface in the target programming language (e.g., R, Python, or Julia), mapping the `FOOTER` to a DataFrame object. Additionally, a `read` method must be implemented on the DataFrame to expose GDAL VFS access (Figure\~\ref{fig\:api\_reader}). Optional helper functions can also be included to perform sanity checks and validate file compliance with the TACO format specification.

<p>&nbsp;</p>
<a name="fig7"></a>
<p align="center">
  <img src="https://github.com/user-attachments/assets/ac910f8e-84d8-4b1a-bcbe-d46dba349692" width="80%">
</p>
<sub><strong>Figure 7:</strong> Overview of the TACO Reader API. This diagram illustrates the core components and their interactions. The Reader parses the FOOTER of TACO and TORTILLA objects and converts them into a DataFrame. Individual SAMPLEs can then be accessed using the read method, which enables sample-level querying and downstream analysis.</sub>
<p>&nbsp;</p>

## Extensions

### Sample-level Semantic Description

#### STAC extension

This section describes the integration of SpatioTemporal Asset Catalog (STAC) metadata at the item level, where each `SAMPLE` corresponds to a STAC Item. STAC provides a standardized schema for spatially and temporally contextualizing assets. Although our schema does not adopt the exact naming conventions defined in official STAC, the current `SAMPLE` STAC extension allows for a direct mapping between the two specifications.

<table>
  <thead>
    <tr>
      <th><strong>Field</strong></th>
      <th><strong>Type</strong></th>
      <th><strong>Details</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>stac:crs</code></td>
      <td>String</td>
      <td><strong>CORE</strong>. The Coordinate Reference System (CRS), specified using a recognized authority (EPSG, ESRI or SR-ORG).</td>
    </tr>
    <tr>
      <td><code>stac:geotransform</code></td>
      <td>Array of Floats</td>
      <td><strong>CORE</strong>. A 6-element array defining the affine transformation from pixel to spatial coordinates, following GDAL conventions:  
        <ul>
          <li><code>a</code>: Top-left x-coordinate of the upper-left pixel</li>
          <li><code>b</code>: Pixel width (x-resolution)</li>
          <li><code>c</code>: Row rotation (usually 0)</li>
          <li><code>d</code>: Top-left y-coordinate of the upper-left pixel</li>
          <li><code>e</code>: Column rotation (usually 0)</li>
          <li><code>f</code>: Negative pixel height (y-resolution, negative for north-up)</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td><code>stac:tensor_shape</code></td>
      <td>Array of integers</td>
      <td><strong>CORE</strong>. The spatial dimensions of the sample.</td>
    </tr>
    <tr>
      <td><code>stac:time_start</code></td>
      <td>Integer</td>
      <td><strong>CORE</strong>. Timestamp in seconds since UNIX epoch, representing the nominal start of acquisition.</td>
    </tr>
    <tr>
      <td><code>stac:time_end</code></td>
      <td>Integer</td>
      <td><strong>CORE</strong>. Timestamp marking the end of the acquisition or composite period.</td>
    </tr>
    <tr>
      <td><code>stac:centroid</code></td>
      <td>String</td>
      <td><strong>AUTOMATIC</strong>. Centroid of the sample in WKT <code>POINT</code> (EPSG:4326).</td>
    </tr>
  </tbody>
</table>

#### RAI extension

The RAI (Responsible AI) extension automatically enriches each `SAMPLE` with socioeconomic and environmental indicators by spatially overlaying its footprint with global datasets.

<table>
  <thead>
    <tr>
      <th><strong>Field</strong></th>
      <th><strong>Type</strong></th>
      <th><strong>Details</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>rai:elevation</code></td>
      <td>Long</td>
      <td><strong>AUTOMATIC</strong>. Average elevation in meters within the Sample footprint (from <a href="https://doi.org/10.5069/G9028PQB">Copernicus DEM</a>).</td>
    </tr>
    <tr>
      <td><code>rai:cisi</code></td>
      <td>Float</td>
      <td><strong>AUTOMATIC</strong>. Critical Infrastructure Spatial Index (0–1). See <a href="https://doi.org/10.1038/s41597-022-01218-4">doi:10.1038/s41597-022-01218-4</a>.</td>
    </tr>
    <tr>
      <td><code>rai:gdp</code></td>
      <td>Float</td>
      <td><strong>AUTOMATIC</strong>. GDP (USD/year) averaged over footprint. See <a href="https://doi.org/10.1038/sdata.2018.4">doi:10.1038/sdata.2018.4</a>.</td>
    </tr>
    <tr>
      <td><code>rai:hdi</code></td>
      <td>Float</td>
      <td><strong>AUTOMATIC</strong>. Human Development Index (0–1). See <a href="https://doi.org/10.1038/sdata.2018.4">doi:10.1038/sdata.2018.4</a>.</td>
    </tr>
    <tr>
      <td><code>rai:gmi</code></td>
      <td>Float</td>
      <td><strong>AUTOMATIC</strong>. Global human modification index. See <a href="https://doi.org/10.5194/essd-12-1953-2020">doi:10.5194/essd-12-1953-2020</a>.</td>
    </tr>
    <tr>
      <td><code>rai:pop</code></td>
      <td>Float</td>
      <td><strong>AUTOMATIC</strong>. Estimated population (LandScan). See <a href="https://doi.org/10.48690/1531770">doi:10.48690/1531770</a>.</td>
    </tr>
    <tr>
      <td><code>rai:admin0</code></td>
      <td>String</td>
      <td><strong>AUTOMATIC</strong>. Country-level boundary. See <a href="https://doi.org/10.1371/journal.pone.0231866">doi:10.1371/journal.pone.0231866</a>.</td>
    </tr>
    <tr>
      <td><code>rai:admin1</code></td>
      <td>String</td>
      <td><strong>AUTOMATIC</strong>. District-level boundary. Same source as above.</td>
    </tr>
    <tr>
      <td><code>rai:admin2</code></td>
      <td>String</td>
      <td><strong>AUTOMATIC</strong>. Municipality-level boundary. Same source as above.</td>
    </tr>
  </tbody>
</table>

#### STATS extension

The STATS extension provides descriptive statistics summarizing the pixel values of each `SAMPLE`. These statistics are computed automatically by the TACO API when the `file_format` is set to `Gtiff`, and they are calculated per band across the spatial dimensions (height × width) of the image. This extension defines four fields: `stats:mean`, `stats:min`, `stats:max`, and `stats:std`. Each field is represented as an array of scalars, with one value per channel. These statistics are essential for tasks such as input normalization, quality assessment, and characterization of value distributions across heterogeneous datasets. Importantly, when all samples in a TORTILLA archive include STATS metadata, the TACO API enables users to compute global or subset-level statistics through pooled variance and weighted averages, without requiring the entire dataset to be loaded into memory.

#### STATS Fields

| Field        | Type            | Description                                                                                |
| ------------ | --------------- | ------------------------------------------------------------------------------------------ |
| `stats:mean` | Array of Floats | The mean value of each band, computed across the height × width spatial dimensions.        |
| `stats:min`  | Array of Floats | The minimum value of each band across the image.                                           |
| `stats:max`  | Array of Floats | The maximum value of each band across the image.                                           |
| `stats:std`  | Array of Floats | The standard deviation of each band, reflecting the variability across the spatial extent. |


### TACO-level Semantic Description

#### Extent Class

Describes the spatial and temporal coverage of the entire dataset. Both spatial and temporal extents are required.

| **Field**  | **Type**         | **Details**                                                              |
| ---------- | ---------------- | ------------------------------------------------------------------------ |
| `spatial`  | List of numbers  | Bounding box defined as `[xmin, ymin, xmax, ymax]` in EPSG:4326.         |
| `temporal` | List of integers | Start and end dates in milliseconds since Unix Epoch (Jan 1, 1970, UTC). |

#### Contact Class

The **Contact class** is based on the [STAC Extension](https://github.com/stac-extensions/contacts) proposed by Matthias Mohr. It identifies and provides contact details for a person or organization responsible for a resource.

| **Field**      | **Type**             | **Details**                                                                   |
| -------------- | -------------------- | ----------------------------------------------------------------------------- |
| `name`         | String               | Name of the responsible person (if `organization` is missing).                |
| `organization` | String               | Affiliation of the contact (if `name` is missing).                            |
| `emails`       | List of Info Objects | Optional email addresses.                                                     |
| `roles`        | List of strings      | Optional roles (duties, functions, permissions) associated with this contact. |

#### Hyperlink Object

The Hyperlink class defines a URL and its associated description. The URL must follow [RFC 3986](https://www.rfc-editor.org/rfc/rfc3986) standards.

| **Field**     | **Type** | **Details**                                          |
| ------------- | -------- | ---------------------------------------------------- |
| `href`        | String   | URL of the resource. Must be a valid URI (RFC 3986). |
| `description` | String   | Optional explanation or context for the hyperlink.   |

#### Task Extension

The `task` field must be a string selected from a well-defined and consistent list of supported ML tasks. It defines the primary ML task that the dataset supports.

### Task Field

| **Field** | **Type**         | **Details**                    |
| --------- | ---------------- | ------------------------------ |
| `task`    | String (Literal) | Type of machine learning task. |

The task field must be one of the following values:

* **Regression**: Estimates a numeric and continuous value.
* **Classification**: Assigns predefined class labels to an output.
* **Scene Classification**: Assigns a single class label to an entire scene or area.
* **Object Detection**: Identifies and localizes objects using bounding boxes.
* **Segmentation**: Labels individual pixels in an image.
* **Semantic Segmentation**: Pixel-wise classification without object differentiation.
* **Instance Segmentation**: Labels each distinct object at the pixel level.
* **Panoptic Segmentation**: Merges semantic and instance segmentation.
* **Similarity Search**: Checks if a query matches any reference item.
* **Generative**: Produces synthetic data.
* **Image Captioning**: Generates textual descriptions of images.
* **Super Resolution**: Enhances image resolution and detail.
* **Denoising**: Removes noise artifacts.
* **Inpainting**: Reconstructs missing/corrupt regions.
* **Colorization**: Adds color to grayscale images.
* **Style Transfer**: Transfers style from one image to another.
* **Deblurring**: Removes blur from an image.
* **Dehazing**: Removes haze/fog to enhance clarity.
* **General**: Use only if no specific task applies; clarify as needed.


#### Split Strategy Extension

The core `split_strategy` field is a string that **must** be chosen from a predefined list of supported splitting approaches. This field details how the dataset is partitioned into distinct subsets, typically for training, validation, and testing machine learning models.

| **Field**        | **Type**         | **Details**                           |
| ---------------- | ---------------- | ------------------------------------- |
| `split_strategy` | String (Literal) | The method used to split the dataset. |

**Supported `split_strategy` values:**

* **random**: The dataset is split into training, validation, and testing subsets through a randomized process.
* **stratified**: The dataset is split while preserving the distribution of a specific property, such as temporal periods (e.g., splitting by year or season) or spatial characteristics (e.g., by geographic location).
* **other**: The dataset is split using a custom or non-standard method. Additional description is recommended.
* **none**: The dataset is not explicitly divided into subsets.
* **unknown**: The method used to split the dataset is not known or unspecified.

#### Sensor Extension

The Sensor extension provides information about the optical remote sensing data, including the sensor used and the spectral bands available. Users can specify the sensor name (e.g., `landsat8oli`, `sentinel2msi`) and optionally select a subset of bands (e.g., `landsat8oli[B01, B02]`). If recognized, the TACO API automatically populates the corresponding bands.

| **Field** | **Type**             | **Details**                                                                                                                                                                                                                                                         |
| --------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `sensor`  | String               | The sensor that acquired the data (optional). **Supported sensors:** `landsat1mss`, `landsat2mss`, `landsat3mss`, `landsat4mss`, `landsat5mss`, `landsat4tm`, `landsat5tm`, `landsat7etm`, `landsat8oli`, `landsat9oli`, `sentinel2msi`, `eo1ali`, `aster`, `modis` |
| `bands`   | List of SpectralBand | A list of spectral band objects. If not provided directly, it will be inferred from the `sensor` field if recognized.                                                                                                                                               |

#### Spectral Band Extension

The spectral band extension describes characteristics of individual bands associated with a given sensor.

| **Field**             | **Type** | **Details**                                                                    |
| --------------------- | -------- | ------------------------------------------------------------------------------ |
| `name`                | String   | Unique name of the band (e.g., "B02", "red") (**required**)                    |
| `index`               | Integer  | Index of the band                                                              |
| `common_name`         | String   | Common name (e.g., "blue", "green") (optional)                                 |
| `description`         | String   | Description of the band (optional)                                             |
| `unit`                | String   | Unit of measurement (optional)                                                 |
| `center_wavelength`   | Float    | Central wavelength of the band (optional)                                      |
| `full_width_half_max` | Float    | Full width at half maximum (FWHM), a measure of spectral resolution (optional) |

#### Label Extension

The Label extension defines label data in a dataset. A `Label` object includes a list of `LabelClass` objects.

| **Field**           | **Type**                   | **Details**                                                    |
| ------------------- | -------------------------- | -------------------------------------------------------------- |
| `label_classes`     | List of LabelClass Objects | A list where each element defines a label class (**required**) |
| `label_description` | String                     | An optional description of the labels used                     |

#### Label Class Extension

Each `LabelClass` defines a specific category or class in the dataset.

| **Field**     | **Type**          | **Details**                                                         |
| ------------- | ----------------- | ------------------------------------------------------------------- |
| `name`        | String            | Unique human-readable name (e.g., "car", "building") (**required**) |
| `category`    | String or Integer | A broader category the label belongs to (**required**)              |
| `description` | String            | Optional detailed description                                       |

#### Scientific Extension

This extension standardizes links to related scientific publications. The TACO scientific extension is based on the [STAC Scientific Extension](https://github.com/stac-extensions/scientific).

| **Field**      | **Type**                    | **Details**                                                                              |
| -------------- | --------------------------- | ---------------------------------------------------------------------------------------- |
| `doi`          | String                      | Digital Object Identifier (DOI) of the dataset                                           |
| `citation`     | String                      | Full BibTeX citation                                                                     |
| `summary`      | String                      | Brief dataset summary                                                                    |
| `publications` | List of Publication Objects | A list of related scientific works, conforming to the `Publication Object` specification |

#### Publication Extension

The `Publication` object contains metadata for a scientific publication related to the dataset.

| **Field**  | **Type** | **Details**                                           |
| ---------- | -------- | ----------------------------------------------------- |
| `doi`      | String   | DOI of the publication (**required**)                 |
| `citation` | String   | Full BibTeX citation (**required**)                   |
| `summary`  | String   | Summary or abstract of the publication (**required**) |


### Facilitating dataset streaming with TOGs

Since version `0.0.2`, TACO supports fully streamable datasets, which is particularly advantageous for nested datasets—specifically, when TORTILLA files are used as the underlying data source. Streaming capabilities in TACO allow users to avoid copying data to local disk by enabling on-demand reading, thereby enhancing performance in cloud-based workflows. This approach significantly reduces the number of HTTP GET requests typically required when accessing individual samples. In non-streaming scenarios, inspecting each sample in a nested dataset results in a separate Parquet read per sample, which in cloud environments leads to additional HTTP requests, increasing both latency and cost.

To mitigate this issue, TACO now embeds all metadata, including both top-level sample metadata and fine-grained chunking details, directly into the FOOTER. As illustrated in Figure \ref{fig\:streamable\_taco}, the FOOTER is organized as a sequence of \$n + 2\$ Parquet files, where \$n\$ is the maximum nesting depth (with \$n = 0\$ for non-nested datasets). The additional two files include a top-level Parquet file that stores sample-level metadata and a bottom-level Parquet file dedicated to chunking-level metadata. The latter is supported only when the underlying data is stored as TACO-Optimized GeoTIFFs (TOGs).

A TOG is a GDAL-compatible GeoTIFF file created using specific compression and tiling options that make it suitable for machine learning workflows. The required creation parameters include `driver="COG"`, `tiled=True`, `interleave="tile"`, `blocksize=512` (divisible by 16 and tunable to the desired patch size), `compress="zstd"`, `level=13`, `bigtiff="yes"`, `overview="none"`, and `predictor="yes"`. Each TOG must embed image-level statistics into the `TIFFTAG_GDAL_METADATA` (ID 42112), specifically under the `IMAGE_INFO` key. These statistics are generated by running the following command:

```
gdalinfo -hist -approx_stats -json -nofl
```

The FOOTER’s offset and length values, stored in the file HEADER, point to the location of the first Parquet file. Each Parquet file contains file-level key-value metadata, encoded as a JSON string, using the key `"pointer"` and the format `OFFSETLENGTH(A, B)` where `A` and `B` represent the byte offset and length of the subsequent Parquet file. This process continues until the `"pointer"` is set to `NULL`, signaling the end of the metadata chain. Every Parquet file includes a column named `root`, which functions as a primary key for maintaining the relational structure across the different levels.
