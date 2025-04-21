Serenity Dashboard System
=========================

Complete Implementation Guide
-----------------------------

Version 1.0

* * *

Table of Contents
=================

1.  Introduction
    *   Overview
    *   Features
    *   Requirements
2.  Installation & Setup
    *   Package Installation
    *   Database Setup
    *   Initial Configuration
3.  Core Components
    *   Dashboard Models
    *   Widget System
    *   Grid Integration
    *   MVC Integration
4.  Implementation Guide
    *   Step-by-Step Setup
    *   Component Creation
    *   Integration Steps
    *   Configuration Details
5.  User Interface
    *   Dashboard Layout
    *   Widget Types
    *   Customization Options
6.  API Reference
    *   Service Methods
    *   Extension Points
    *   Event Handlers
7.  Advanced Features
    *   Custom Widgets
    *   Data Sources
    *   Security Integration
8.  Troubleshooting
    *   Common Issues
    *   Solutions
    *   Best Practices
9.  Appendices
    *   Code Examples
    *   Configuration Templates
    *   Migration Guide

* * *

1. Introduction
===============

Overview
--------

The Serenity Dashboard System provides a flexible and extensible framework for creating dynamic dashboards within Serenity applications. It allows users to create widgets from existing grids and visualize data through various chart types and tables.

Features
--------

*   Dynamic widget creation from grids
*   Multiple visualization types
*   Drag-and-drop interface
*   Real-time data updates
*   Customizable layouts
*   Integrated security

Requirements
------------

*   Serenity.NET Platform 6.0+
*   SQL Server 2016+
*   Node.js 14+
*   Modern web browser

* * *

2. Installation & Setup
=======================

2.1 Package Installation
------------------------

### NPM Dependencies

    {
        "dependencies": {
            "gridstack": "^5.0.0",
            "chart.js": "^3.7.0",
            "@serenity-is/corelib": "^6.0.0",
            "@types/react": "^17.0.0",
            "@types/jquery": "^3.5.0"
        }
    }
    

### NuGet Packages

    <ItemGroup>
        <PackageReference Include="Serenity.Web" Version="6.0.0" />
        <PackageReference Include="Serenity.Scripts" Version="6.0.0" />
    </ItemGroup>
    

2.2 Database Setup
------------------

    -- Create Dashboard Tables
    CREATE TABLE Dashboards (
        Id INT IDENTITY(1,1) PRIMARY KEY,
        Name NVARCHAR(100) NOT NULL,
        Description NVARCHAR(500),
        IsDefault BIT NOT NULL DEFAULT 0,
        UserId INT NOT NULL,
        Layout NVARCHAR(MAX)
    );
    
    CREATE TABLE Widgets (
        Id INT IDENTITY(1,1) PRIMARY KEY,
        Name NVARCHAR(100) NOT NULL,
        Type NVARCHAR(50) NOT NULL,
        Config NVARCHAR(MAX),
        Position NVARCHAR(MAX),
        DashboardId INT NOT NULL,
        SourceGrid NVARCHAR(200),
        AutoRefresh INT,
        FOREIGN KEY (DashboardId) REFERENCES Dashboards(Id)
    );
    

2.3 Initial Configuration
-------------------------

### Startup.cs

    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddDashboardSystem();
        }
    
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            app.UseDashboardSystem();
        }
    }
    

* * *

3. Core Components
==================

3.1 Dashboard Models
--------------------

### DashboardRow.cs

    public class DashboardRow : Row<DashboardRow.RowFields>
    {
        [IdProperty]
        public int? Id { get; set; }
        
        [Required, StringLength(100)]
        public string Name { get; set; }
        
        [StringLength(500)]
        public string Description { get; set; }
        
        public bool? IsDefault { get; set; }
        
        [Required]
        public int? UserId { get; set; }
        
        public string Layout { get; set; }
    
        public class RowFields : RowFieldsBase
        {
            public Int32Field Id;
            public StringField Name;
            public StringField Description;
            public BooleanField IsDefault;
            public Int32Field UserId;
            public StringField Layout;
        }
    }
    

### WidgetRow.cs

    public class WidgetRow : Row<WidgetRow.RowFields>
    {
        [IdProperty]
        public int? Id { get; set; }
        
        [Required, StringLength(100)]
        public string Name { get; set; }
        
        [Required, StringLength(50)]
        public string Type { get; set; }
        
        public string Config { get; set; }
        
        public string Position { get; set; }
        
        [Required]
        public int? DashboardId { get; set; }
        
        [StringLength(200)]
        public string SourceGrid { get; set; }
        
        public int? AutoRefresh { get; set; }
    
        public class RowFields : RowFieldsBase
        {
            public Int32Field Id;
            public StringField Name;
            public StringField Type;
            public StringField Config;
            public StringField Position;
            public Int32Field DashboardId;
            public StringField SourceGrid;
            public Int32Field AutoRefresh;
        }
    }
    

3.2 MVC Integration
-------------------

### Dashboard.cshtml

    @{
        ViewData["Title"] = "Dashboard";
    }
    
    @section Head {
        @Html.StyleBundle("Pages/Dashboard")
    }
    
    <div id="DashboardDiv"></div>
    
    @section Scripts {
        @Html.ScriptBundle("Pages/Dashboard")
        <script type="text/javascript">
            jQuery(function () {
                new YourApp.Dashboard.DashboardPage({
                    element: '#DashboardDiv'
                });
            });
        </script>
    }
    

### DashboardController.cs

    [RoutePrefix("Dashboard"), Route("{action=index}")]
    public class DashboardController : Controller
    {
        private readonly IServiceProvider _serviceProvider;
    
        public DashboardController(IServiceProvider serviceProvider)
        {
            _serviceProvider = serviceProvider;
        }
    
        [HttpGet]
        public ActionResult Index()
        {
            return View("~/Views/Dashboard/Dashboard.cshtml");
        }
    
        [HttpPost]
        public async Task<JsonResult> SaveLayout([FromBody] SaveLayoutRequest request)
        {
            // Implementation
        }
    }
    

3.3 Dashboard Page Component
----------------------------

    @Decorators.registerClass('YourApp.Dashboard.DashboardPage')
    export class DashboardPage extends React.Component<{}, DashboardPageState> {
        private grid: GridStack;
        private readonly dashboardService = new DashboardService();
    
        state: DashboardPageState = {
            dashboards: [],
            currentDashboard: null,
            widgets: [],
            isEditMode: false
        };
    
        async componentDidMount() {
            await this.loadDashboard();
            this.initializeGrid();
        }
    
        private initializeGrid() {
            const gridElement = document.querySelector('.grid-stack');
            if (!gridElement) return;
    
            this.grid = GridStack.init({
                column: 12,
                cellHeight: 80,
                animate: true
            });
    
            this.renderWidgets();
        }
    
        private renderWidgets() {
            const { widgets } = this.state;
            
            widgets.forEach(widget => {
                const widgetElement = this.createWidgetElement(widget);
                this.grid.addWidget(widgetElement, widget.position);
            });
        }
    
        render() {
            const { currentDashboard, isEditMode } = this.state;
    
            return (
                <div className="s-dashboard-page">
                    <div className="toolbar">
                        <div className="dashboard-title">
                            <h2>{currentDashboard?.name}</h2>
                        </div>
                        <div className="toolbar-actions">
                            <button 
                                className="btn btn-primary"
                                onClick={() => this.addWidget()}
                            >
                                Add Widget
                            </button>
                            <button 
                                className={`btn ${isEditMode ? 'btn-success' : 'btn-default'}`}
                                onClick={this.toggleEditMode}
                            >
                                {isEditMode ? 'Save Layout' : 'Edit Layout'}
                            </button>
                        </div>
                    </div>
                    <div className="grid-stack"></div>
                </div>
            );
        }
    }
    

3.4 Grid Widget Creation
------------------------

    @Decorators.registerClass('Common.CreateWidgetDialog')
    export class CreateWidgetDialog extends TemplatedDialog<CreateWidgetOptions> {
        private form: CreateWidgetForm;
        private grid: EntityGrid<any, any>;
        private gridAnalysis: GridAnalysis;
    
        constructor(options: CreateWidgetOptions) {
            super(options);
            this.grid = options.grid;
            this.gridAnalysis = GridAnalyzer.analyze(this.grid);
            this.initializeDialog();
        }
    
        protected getTemplate(): string {
            return `
    <div class="s-Form">
        <form id="~_Form" action="">
            <div class="field">
                <label for="~_WidgetName" class="caption">Widget Name</label>
                <input id="~_WidgetName" type="text" class="required" />
            </div>
            <div class="field">
                <label for="~_WidgetType" class="caption">Widget Type</label>
                <select id="~_WidgetType">
                    <option value="table">Table</option>
                    <option value="chart">Chart</option>
                </select>
            </div>
            <div id="~_ChartSection">
                <div class="field">
                    <label for="~_ChartType" class="caption">Chart Type</label>
                    <select id="~_ChartType"></select>
                </div>
            </div>
        </form>
    </div>`;
        }
    
        private async createWidget() {
            if (!this.validateForm()) return;
    
            const config = this.buildWidgetConfig();
            
            try {
                await WidgetService.Create({
                    Entity: {
                        name: this.form.Name.value,
                        type: this.form.Type.value,
                        config: JSON.stringify(config),
                        dashboardId: this.options.dashboardId
                    }
                });
    
                this.dialogClose();
                Q.notifySuccess('Widget created successfully');
            }
            catch (e) {
                Q.notifyError(e);
            }
        }
    }
    

4. Implementation Guide (Continued)
===================================

4.1 Widget System Implementation
--------------------------------

### 4.1.1 Base Widget Class

    abstract class BaseWidget implements IWidget {
        protected element: HTMLElement;
        protected config: WidgetConfig;
        protected data: any;
    
        constructor(element: HTMLElement, config: WidgetConfig) {
            this.element = element;
            this.config = config;
        }
    
        abstract render(): void;
        
        async refresh(): Promise<void> {
            await this.loadData();
            this.render();
        }
    
        protected async loadData(): Promise<void> {
            try {
                const response = await ServiceRequest.execute({
                    url: this.config.dataSource,
                    request: this.buildRequest()
                });
                this.data = response;
            } catch (error) {
                Q.notifyError("Failed to load widget data");
            }
        }
    
        protected buildRequest(): any {
            return {
                Criteria: this.config.criteria,
                IncludeColumns: this.config.columns
            };
        }
    }
    

### 4.1.2 Table Widget Implementation

    @Decorators.registerClass('Dashboard.Widgets.TableWidget')
    export class TableWidget extends BaseWidget {
        private grid: DataGrid<any>;
    
        render(): void {
            const element = document.createElement('div');
            this.element.appendChild(element);
    
            this.grid = new DataGrid({
                element: element,
                dataSource: this.data,
                columns: this.getColumns(),
                options: {
                    enableColumnReorder: false,
                    multiSelect: false,
                    rowHeight: 30,
                    autoHeight: true
                }
            });
        }
    
        private getColumns(): Column[] {
            return this.config.columns.map(col => ({
                field: col.field,
                name: col.name,
                width: col.width,
                format: col.format,
                sortable: true
            }));
        }
    }
    

### 4.1.3 Chart Widget Implementation

    @Decorators.registerClass('Dashboard.Widgets.ChartWidget')
    export class ChartWidget extends BaseWidget {
        private chart: Chart;
    
        render(): void {
            const canvas = document.createElement('canvas');
            this.element.appendChild(canvas);
    
            const ctx = canvas.getContext('2d');
            if (!ctx) return;
    
            this.chart = new Chart(ctx, {
                type: this.config.chartType,
                data: this.prepareChartData(),
                options: this.getChartOptions()
            });
        }
    
        private prepareChartData(): ChartData {
            switch (this.config.chartType) {
                case 'pie':
                    return this.preparePieData();
                case 'line':
                    return this.prepareLineData();
                case 'bar':
                    return this.prepareBarData();
                default:
                    throw new Error(`Unsupported chart type: ${this.config.chartType}`);
            }
        }
    
        private getChartOptions(): ChartOptions {
            return {
                responsive: true,
                maintainAspectRatio: false,
                animation: {
                    duration: 500
                },
                plugins: {
                    legend: {
                        position: 'top'
                    },
                    title: {
                        display: true,
                        text: this.config.title
                    }
                }
            };
        }
    }
    

4.2 Dashboard Layout Implementation
-----------------------------------

### 4.2.1 Layout Manager

    class DashboardLayoutManager {
        private grid: GridStack;
        private readonly dashboardService: DashboardService;
        private readonly widgets: Map<string, BaseWidget>;
    
        constructor(element: HTMLElement, options: DashboardOptions) {
            this.widgets = new Map();
            this.initializeGrid(element, options);
        }
    
        private initializeGrid(element: HTMLElement, options: DashboardOptions): void {
            this.grid = GridStack.init({
                element: element,
                column: 12,
                cellHeight: 80,
                animate: true,
                draggable: {
                    handle: '.widget-header'
                },
                resizable: {
                    handles: 'e,se,s,sw,w'
                },
                ...options
            });
    
            this.grid.on('change', this.handleLayoutChange.bind(this));
            this.grid.on('resize', this.handleWidgetResize.bind(this));
        }
    
        async addWidget(config: WidgetConfig): Promise<void> {
            const widget = this.createWidget(config);
            const element = this.createWidgetElement(config);
            
            this.grid.addWidget(element, {
                x: config.position?.x || 0,
                y: config.position?.y || 0,
                w: config.position?.width || 3,
                h: config.position?.height || 2
            });
    
            this.widgets.set(config.id, widget);
            await widget.refresh();
        }
    
        private createWidgetElement(config: WidgetConfig): HTMLElement {
            const element = document.createElement('div');
            element.className = 'grid-stack-item';
            element.innerHTML = `
                <div class="widget">
                    <div class="widget-header">
                        <span class="widget-title">${Q.htmlEncode(config.title)}</span>
                        <div class="widget-actions">
                            <button class="refresh-btn"><i class="fa fa-refresh"></i></button>
                            <button class="edit-btn"><i class="fa fa-cog"></i></button>
                            <button class="remove-btn"><i class="fa fa-times"></i></button>
                        </div>
                    </div>
                    <div class="widget-content"></div>
                </div>
            `;
    
            this.attachWidgetEventHandlers(element, config);
            return element;
        }
    
        private attachWidgetEventHandlers(element: HTMLElement, config: WidgetConfig): void {
            const refreshBtn = element.querySelector('.refresh-btn');
            refreshBtn?.addEventListener('click', () => this.refreshWidget(config.id));
    
            const editBtn = element.querySelector('.edit-btn');
            editBtn?.addEventListener('click', () => this.editWidget(config.id));
    
            const removeBtn = element.querySelector('.remove-btn');
            removeBtn?.addEventListener('click', () => this.removeWidget(config.id));
        }
    
        async saveLayout(): Promise<void> {
            const layout = this.grid.save();
            await this.dashboardService.saveLayout({
                DashboardId: this.options.dashboardId,
                Layout: JSON.stringify(layout)
            });
        }
    }
    

4.3 Data Integration
--------------------

### 4.3.1 Widget Data Service

    class WidgetDataService {
        async fetchData(config: WidgetConfig): Promise<any> {
            const request = this.buildRequest(config);
            
            switch (config.dataSource.type) {
                case 'grid':
                    return await this.fetchGridData(config.dataSource.gridId, request);
                case 'service':
                    return await this.fetchServiceData(config.dataSource.url, request);
                case 'custom':
                    return await this.fetchCustomData(config.dataSource.handler, request);
                default:
                    throw new Error(`Unsupported data source type: ${config.dataSource.type}`);
            }
        }
    
        private buildRequest(config: WidgetConfig): ListRequest {
            return {
                Criteria: config.criteria,
                IncludeColumns: config.columns,
                Sort: config.sort,
                Skip: config.skip,
                Take: config.take
            };
        }
    
        private async fetchGridData(gridId: string, request: ListRequest): Promise<ListResponse<any>> {
            const grid = Q.getGridById(gridId);
            if (!grid) throw new Error(`Grid not found: ${gridId}`);
            
            return await grid.getList(request);
        }
    }
    

### 4.3.2 Data Transformation

    class DataTransformer {
        static transformForChart(data: any[], config: ChartConfig): ChartData {
            switch (config.chartType) {
                case 'pie':
                    return this.transformForPieChart(data, config);
                case 'line':
                    return this.transformForLineChart(data, config);
                case 'bar':
                    return this.transformForBarChart(data, config);
                default:
                    throw new Error(`Unsupported chart type: ${config.chartType}`);
            }
        }
    
        private static transformForPieChart(data: any[], config: ChartConfig): ChartData {
            const labels = data.map(item => item[config.labelField]);
            const values = data.map(item => item[config.valueField]);
    
            return {
                labels: labels,
                datasets: [{
                    data: values,
                    backgroundColor: this.generateColors(values.length)
                }]
            };
        }
    
        private static generateColors(count: number): string[] {
            // Implementation of color generation
            return [];
        }
    }
    

4.4 Security Implementation
---------------------------

### 4.4.1 Dashboard Permissions

    public static class DashboardPermissions
    {
        public const string View = "Dashboard:View";
        public const string Create = "Dashboard:Create";
        public const string Update = "Dashboard:Update";
        public const string Delete = "Dashboard:Delete";
        
        public static readonly string[] All = {
            View, Create, Update, Delete
        };
    }
    
    public class DashboardPermissionAttribute : PermissionAttributeBase
    {
        public DashboardPermissionAttribute(string permission) 
            : base(permission)
        {
        }
    }
    

### 4.4.2 Security Handlers

    public class DashboardSecurityHandler
    {
        private readonly IUserAccessor userAccessor;
        private readonly IPermissionService permissions;
    
        public DashboardSecurityHandler(
            IUserAccessor userAccessor,
            IPermissionService permissions)
        {
            this.userAccessor = userAccessor;
            this.permissions = permissions;
        }
    
        public bool CanAccessDashboard(int dashboardId)
        {
            if (!permissions.HasPermission(DashboardPermissions.View))
                return false;
    
            var dashboard = // Load dashboard
            return dashboard.UserId == userAccessor.User?.GetIdentifier();
        }
    
        public bool CanModifyWidget(int widgetId)
        {
            if (!permissions.HasPermission(DashboardPermissions.Update))
                return false;
    
            var widget = // Load widget
            return CanAccessDashboard(widget.DashboardId);
        }
    }
    

4.5 Styling Implementation
--------------------------

### 4.5.1 Dashboard Styles

    .s-dashboard-page {
        display: flex;
        flex-direction: column;
        height: 100vh;
        
        .toolbar {
            padding: 8px;
            background: #f5f5f5;
            border-bottom: 1px solid #ddd;
            display: flex;
            justify-content: space-between;
            align-items: center;
            
            .tool-button {
                margin-right: 6px;
            }
        }
    
        .grid-stack {
            flex: 1;
            background: #fafafa;
            padding: 16px;
            
            .grid-stack-item {
                .widget {
                    height: 100%;
                    background: white;
                    border: 1px solid #ddd;
                    border-radius: 4px;
                    box-shadow: 0 1px 3px rgba(0,0,0,0.1);
                    display: flex;
                    flex-direction: column;
                    
                    .widget-header {
                        padding: 8px;
                        border-bottom: 1px solid #ddd;
                        background: #f8f8f8;
                        display: flex;
                        justify-content: space-between;
                        align-items: center;
                        cursor: move;
                        
                        .widget-title {
                            font-weight: 600;
                            flex: 1;
                        }
                        
                        .widget-actions {
                            display: flex;
                            gap: 4px;
                            
                            button {
                                padding: 4px;
                                background: none;
                                border: none;
                                cursor: pointer;
                                opacity: 0.6;
                                
                                &:hover {
                                    opacity: 1;
                                }
                            }
                        }
                    }
                    
                    .widget-content {
                        flex: 1;
                        padding: 8px;
                        overflow: auto;
                    }
                }
            }
        }
    }
    
    .s-CreateWidgetDialog {
        width: 600px;
        
        .form {
            padding: 16px;
            
            .field {
                margin-bottom: 16px;
                
                label {
                    display: block;
                    margin-bottom: 4px;
                    font-weight: 500;
                }
                
                input, select {
                    width: 100%;
                    padding: 6px 8px;
                    border: 1px solid #ddd;
                    border-radius: 3px;
                    
                    &:focus {
                        border-color: #66afe9;
                        outline: none;
                    }
                }
            }
        }
    }
    
