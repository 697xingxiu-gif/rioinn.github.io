#<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>优惠券批次管理系统</title>
    
    <!-- 引入 Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- 引入 React 和 ReactDOM -->
    <script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
    
    <!-- 引入 Babel 用于解析 JSX -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif; }
    </style>
</head>
<body class="bg-gray-50">
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useMemo, useEffect } = React;

        // --- 图标组件 ---
        const IconBase = ({ size = 24, className = "", children }) => (
            <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={className}>{children}</svg>
        );
        const Plus = (props) => <IconBase {...props}><line x1="12" y1="5" x2="12" y2="19"></line><line x1="5" y1="12" x2="19" y2="12"></line></IconBase>;
        const Edit3 = (props) => <IconBase {...props}><path d="M12 20h9"></path><path d="M16.5 3.5a2.121 2.121 0 0 1 3 3L7 19l-4 1 1-4L16.5 3.5z"></path></IconBase>;
        const Trash2 = (props) => <IconBase {...props}><polyline points="3 6 5 6 21 6"></polyline><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path><line x1="10" y1="11" x2="10" y2="17"></line><line x1="14" y1="11" x2="14" y2="17"></line></IconBase>;
        const CheckCircle = (props) => <IconBase {...props}><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"></path><polyline points="22 4 12 14.01 9 11.01"></polyline></IconBase>;
        const XCircle = (props) => <IconBase {...props}><circle cx="12" cy="12" r="10"></circle><line x1="15" y1="9" x2="9" y2="15"></line><line x1="9" y1="9" x2="15" y2="15"></line></IconBase>;
        const Clock = (props) => <IconBase {...props}><circle cx="12" cy="12" r="10"></circle><polyline points="12 6 12 12 16 14"></polyline></IconBase>;
        const Power = (props) => <IconBase {...props}><path d="M18.36 6.64a9 9 0 1 1-12.73 0"></path><line x1="12" y1="2" x2="12" y2="12"></line></IconBase>;
        const FileText = (props) => <IconBase {...props}><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path><polyline points="14 2 14 8 20 8"></polyline><line x1="16" y1="13" x2="8" y2="13"></line><line x1="16" y1="17" x2="8" y2="17"></line><polyline points="10 9 9 9 8 9"></polyline></IconBase>;
        const AlertTriangle = (props) => <IconBase {...props}><path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"></path><line x1="12" y1="9" x2="12" y2="13"></line><line x1="12" y1="17" x2="12.01" y2="17"></line></IconBase>;
        const Tag = (props) => <IconBase {...props}><path d="M20.59 13.41l-7.17 7.17a2 2 0 0 1-2.83 0L2 12V2h10l8.59 8.59a2 2 0 0 1 0 2.82z"></path><line x1="7" y1="7" x2="7.01" y2="7"></line></IconBase>;
        const RefreshCw = (props) => <IconBase {...props}><polyline points="23 4 23 10 17 10"></polyline><polyline points="1 20 1 14 7 14"></polyline><path d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15"></path></IconBase>;

        // --- 模拟初始数据 ---
        // pendingStock: 用于存储待审核的库存修改值
        const INITIAL_DATA = [
            {
                id: 5,
                batchNo: '2023110501', 
                name: '双11预热草稿',
                amount: 200,
                validityType: 'fixed',
                validityStartDate: '2025-11-01',
                validityEndDate: '2025-11-11',
                validityDays: '',
                scenario: 'general',
                stackable: false,
                ruleType: 'threshold',
                thresholdAmount: 2000,
                status: 'draft', 
                createTime: Date.now(),
                updateTime: Date.now(),
                operator: '运营-小王',
                auditTime: null,
                auditor: '-',
                auditRemark: '',
                totalQuantity: 5000,
                claimedQuantity: 0,
                pendingStock: null
            },
            {
                id: 3,
                batchNo: '2023102588',
                name: '回归用户满减券',
                amount: 100,
                validityType: 'relative',
                validityDays: 15,
                scenario: 'reactivation',
                stackable: false,
                ruleType: 'threshold',
                thresholdAmount: 1000,
                status: 'auditing', 
                createTime: Date.now() - 500000,
                updateTime: Date.now() - 500000,
                operator: '运营-小王',
                auditTime: null,
                auditor: '-',
                auditRemark: '',
                totalQuantity: 2000,
                claimedQuantity: 0,
                pendingStock: null
            },
            {
                id: 1,
                batchNo: '2023061801',
                name: '六月年中大促新人券',
                amount: 50,
                validityType: 'relative',
                validityDays: 7,
                scenario: 'new_user',
                stackable: false,
                ruleType: 'threshold',
                thresholdAmount: 300,
                status: 'approved', 
                createTime: Date.now() - 10000000,
                updateTime: Date.now() - 9000000,
                operator: '运营-小王',
                auditTime: Date.now() - 9000000,
                auditor: '管理员-老李',
                auditRemark: '符合活动要求',
                totalQuantity: 1000,
                claimedQuantity: 450,
                pendingStock: null
            },
            {
                id: 2,
                batchNo: '2023052099',
                name: '社群活跃专属红包',
                amount: 5,
                validityType: 'relative',
                validityDays: 30,
                scenario: 'community',
                stackable: true,
                ruleType: 'none',
                thresholdAmount: 0,
                status: 'approved', 
                createTime: Date.now() - 12000000,
                updateTime: Date.now() - 11000000,
                operator: '运营-小张',
                auditTime: Date.now() - 11000000,
                auditor: '管理员-老李',
                auditRemark: '通过',
                totalQuantity: 500,
                claimedQuantity: 460, 
                pendingStock: 800 // 模拟：有一个待审核的库存修改
            },
            {
                id: 4,
                batchNo: '2023010155',
                name: '限时秒杀通用券',
                amount: 20,
                validityType: 'relative',
                validityDays: 7,
                scenario: 'general',
                stackable: true,
                ruleType: 'none',
                thresholdAmount: 0,
                status: 'approved', 
                createTime: Date.now() - 20000000,
                updateTime: Date.now() - 19000000,
                operator: '运营-小王',
                auditTime: Date.now() - 19000000,
                auditor: '管理员-老李',
                auditRemark: '',
                totalQuantity: 100,
                claimedQuantity: 100,
                pendingStock: null 
            },
            {
                id: 6,
                batchNo: '2023121200',
                name: '违规测试券',
                amount: 1000,
                validityType: 'fixed',
                validityStartDate: '2023-01-01',
                validityEndDate: '2025-12-31',
                validityDays: '',
                scenario: 'new_user',
                stackable: true,
                ruleType: 'none',
                thresholdAmount: 0,
                status: 'rejected', 
                createTime: Date.now() - 3000000,
                updateTime: Date.now() - 2000000,
                operator: '运营-实习生',
                auditTime: Date.now() - 2000000,
                auditor: '管理员-老李',
                auditRemark: '金额过大，有效期过长，不符合规范',
                totalQuantity: 10,
                claimedQuantity: 0,
                pendingStock: null
            },
            {
                id: 7,
                batchNo: '2022020222',
                name: '过期春节活动',
                amount: 88,
                validityType: 'relative',
                validityDays: 7,
                scenario: 'community',
                stackable: true,
                ruleType: 'none',
                thresholdAmount: 0,
                status: 'finished', 
                createTime: Date.now() - 50000000,
                updateTime: Date.now() - 40000000,
                operator: '运营-小王',
                auditTime: Date.now() - 45000000,
                auditor: '管理员-老李',
                auditRemark: '通过',
                totalQuantity: 1000,
                claimedQuantity: 888,
                pendingStock: null
            }
        ];

        // --- 选项配置 ---
        const SCENARIOS = [
            { value: 'community', label: '社群福利' },
            { value: 'new_user', label: '新用户福利' },
            { value: 'reactivation', label: '促活福利' },
            { value: 'general', label: '通用活动' },
        ];

        // --- 主应用组件 ---
        const App = () => {
            const [coupons, setCoupons] = useState(INITIAL_DATA);
            const [isModalOpen, setIsModalOpen] = useState(false);
            const [isAuditModalOpen, setIsAuditModalOpen] = useState(false);
            const [formData, setFormData] = useState({});
            const [auditData, setAuditData] = useState({ id: null, passed: true, remark: '' });

            // 生成10位不重复随机批次号
            const generateBatchNo = (currentCoupons) => {
                let batchNo;
                let isDuplicate = true;
                while (isDuplicate) {
                    batchNo = Math.floor(1000000000 + Math.random() * 9000000000).toString();
                    // eslint-disable-next-line
                    isDuplicate = currentCoupons.some(c => c.batchNo === batchNo);
                }
                return batchNo;
            };

            // 打开新增/编辑弹窗
            const openFormModal = (coupon = null) => {
                if (coupon) {
                    setFormData({ ...coupon });
                } else {
                    setFormData({
                        id: null,
                        name: '',
                        amount: '',
                        validityType: 'relative', 
                        validityDays: '',
                        validityStartDate: '',
                        validityEndDate: '',
                        scenario: 'community',
                        stackable: false,
                        ruleType: 'none',
                        thresholdAmount: '',
                        status: 'draft',
                        createTime: Date.now(),
                        auditRemark: '',
                        totalQuantity: '', 
                        claimedQuantity: 0,
                    });
                }
                setIsModalOpen(true);
            };

            // 打开审核弹窗
            const openAuditModal = (coupon) => {
                setAuditData({ id: coupon.id, passed: true, remark: '' });
                setIsAuditModalOpen(true);
            };

            // 保存优惠券（新增或更新）
            const handleSave = (e) => {
                e.preventDefault();
                
                // 校验有效期
                if (formData.validityType === 'relative' && !formData.validityDays) {
                    alert('请输入有效天数');
                    return;
                }
                if (formData.validityType === 'fixed' && (!formData.validityStartDate || !formData.validityEndDate)) {
                    alert('请选择完整的起止日期');
                    return;
                }

                // 校验：在修改已通过的库存时，不能低于当前库存总数
                if (formData.id) {
                    const originalCoupon = coupons.find(c => c.id === formData.id);
                    if (originalCoupon && originalCoupon.status === 'approved') {
                        if (Number(formData.totalQuantity) < originalCoupon.totalQuantity) {
                            alert(`发放总量不能低于当前库存总数 (${originalCoupon.totalQuantity})`);
                            return;
                        }
                    }
                }

                const now = Date.now();
                const operator = '运营-小王'; // 模拟当前操作人

                if (formData.id) {
                    // 编辑逻辑
                    setCoupons(prev => prev.map(c => {
                        if (c.id === formData.id) {
                            // 特殊处理：如果是已通过状态，仅允许修改库存
                            if (c.status === 'approved') {
                                if (Number(formData.totalQuantity) !== c.totalQuantity) {
                                    // 产生库存修改申请
                                    return {
                                        ...c,
                                        pendingStock: Number(formData.totalQuantity),
                                        updateTime: now,
                                        operator: operator
                                    };
                                }
                                return c; // 无变化
                            }
                            
                            // 其他状态（草稿、未通过），正常修改所有字段
                            return { 
                                ...formData, 
                                status: 'draft',
                                updateTime: now,
                                operator: operator,
                                pendingStock: null 
                            };
                        }
                        return c;
                    }));
                } else {
                    // 新增逻辑
                    const newBatchNo = generateBatchNo(coupons);
                    const newCoupon = {
                        ...formData,
                        id: now,
                        batchNo: newBatchNo, 
                        createTime: now,
                        updateTime: now, 
                        operator: operator, 
                        auditTime: null,
                        auditor: '-',
                        status: 'draft',
                        claimedQuantity: 0,
                        pendingStock: null
                    };
                    setCoupons(prev => [newCoupon, ...prev]);
                }
                setIsModalOpen(false);
            };

            // 提交审核
            const handleSubmitAudit = (id) => {
                if (confirm('确认提交审核吗？提交后将无法编辑。')) {
                    updateStatus(id, 'auditing');
                }
            };

            // 执行审核
            const handleAuditAction = () => {
                const newStatus = auditData.passed ? 'approved' : 'rejected';
                const now = Date.now();
                
                setCoupons(prev => prev.map(c => {
                    if (c.id === auditData.id) {
                        // 场景区分：是库存审核，还是新建审核
                        if (c.pendingStock !== null) {
                            // --- 库存审核 ---
                            if (auditData.passed) {
                                return {
                                    ...c,
                                    totalQuantity: c.pendingStock, // 应用新库存
                                    pendingStock: null, // 清除 pending
                                    auditTime: now,
                                    auditor: '管理员-老李',
                                    updateTime: now,
                                    auditRemark: auditData.remark
                                };
                            } else {
                                return {
                                    ...c,
                                    pendingStock: null, // 仅清除 pending，原库存不变
                                    auditTime: now,
                                    auditor: '管理员-老李',
                                    updateTime: now,
                                    auditRemark: auditData.remark
                                };
                            }
                        } else {
                            // --- 普通新建审核 ---
                            return { 
                                ...c, 
                                status: newStatus, 
                                auditRemark: auditData.remark,
                                auditTime: now, 
                                auditor: '管理员-老李', 
                                updateTime: now 
                            };
                        }
                    }
                    return c;
                }));
                setIsAuditModalOpen(false);
            };

            // 结束活动
            const handleFinish = (id) => {
                if (confirm('确认要结束该批次优惠券活动吗？结束后用户无法领取，前端将显示“活动已结束”。')) {
                    updateStatus(id, 'finished');
                }
            };

            // 删除
            const handleDelete = (id) => {
                if (confirm('确认删除该批次吗？')) {
                    setCoupons(prev => prev.filter(c => c.id !== id));
                }
            };

            // 更新状态通用函数
            const updateStatus = (id, status) => {
                const now = Date.now();
                setCoupons(prev => prev.map(c => c.id === id ? { 
                    ...c, 
                    status,
                    updateTime: now, 
                    operator: '运营-小王'
                } : c));
            };

            // 排序：按创建时间倒序
            const sortedCoupons = useMemo(() => {
                return [...coupons].sort((a, b) => b.createTime - a.createTime);
            }, [coupons]);

            // 获取状态对应的样式和文字
            const getStatusBadge = (status) => {
                const config = {
                    draft: { color: 'bg-gray-100 text-gray-700', text: '待提交', icon: <Edit3 size={14} /> },
                    auditing: { color: 'bg-yellow-100 text-yellow-700', text: '审核中', icon: <Clock size={14} /> },
                    approved: { color: 'bg-green-100 text-green-700', text: '已通过', icon: <CheckCircle size={14} /> },
                    rejected: { color: 'bg-red-100 text-red-700', text: '未通过', icon: <XCircle size={14} /> },
                    finished: { color: 'bg-slate-200 text-slate-500', text: '已结束', icon: <Power size={14} /> },
                };
                const current = config[status] || config.draft;
                return (
                    <span className={`inline-flex items-center gap-1 px-2.5 py-0.5 rounded-full text-xs font-medium ${current.color}`}>
                        {current.icon}
                        {current.text}
                    </span>
                );
            };

            // 格式化场景文字
            const getScenarioLabel = (val) => SCENARIOS.find(s => s.value === val)?.label || val;

            // 判断是否是只读模式 (针对已通过的非库存字段)
            const isApproved = formData.status === 'approved';

            return (
                <div className="min-h-screen bg-gray-50 text-gray-800 font-sans">
                    {/* 顶部导航 */}
                    <header className="bg-white shadow-sm border-b border-gray-200">
                        <div className="w-full max-w-[95%] mx-auto px-4 sm:px-6 lg:px-8 h-16 flex items-center justify-between">
                            <div className="flex items-center gap-2">
                                <div className="bg-blue-600 p-2 rounded-lg">
                                    <FileText className="text-white w-5 h-5" />
                                </div>
                                <h1 className="text-xl font-bold text-gray-900">优惠券批次管理系统</h1>
                            </div>
                            <button 
                                onClick={() => openFormModal()}
                                className="inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500"
                            >
                                <Plus className="w-4 h-4 mr-2" />
                                新建批次
                            </button>
                        </div>
                    </header>

                    {/* 主体内容 */}
                    <main className="w-full max-w-[95%] mx-auto px-4 sm:px-6 lg:px-8 py-8">
                        
                        {/* 数据统计卡片 */}
                        <div className="grid grid-cols-1 md:grid-cols-4 gap-4 mb-6">
                            <div className="bg-white p-4 rounded-lg shadow-sm border border-gray-200">
                                <div className="text-sm text-gray-500">总批次</div>
                                <div className="text-2xl font-bold">{coupons.length}</div>
                            </div>
                            <div className="bg-white p-4 rounded-lg shadow-sm border border-gray-200">
                                <div className="text-sm text-gray-500">运行中活动</div>
                                <div className="text-2xl font-bold text-green-600">
                                    {coupons.filter(c => c.status === 'approved').length}
                                </div>
                            </div>
                            <div className="bg-white p-4 rounded-lg shadow-sm border border-gray-200">
                                <div className="text-sm text-gray-500">待审核</div>
                                <div className="text-2xl font-bold text-yellow-600">
                                    {coupons.filter(c => c.status === 'auditing' || c.pendingStock !== null).length}
                                </div>
                            </div>
                        </div>

                        {/* 列表区域 */}
                        <div className="bg-white shadow overflow-hidden sm:rounded-lg border border-gray-200">
                            <div className="px-4 py-5 sm:px-6 border-b border-gray-200 flex justify-between items-center">
                                <h3 className="text-lg leading-6 font-medium text-gray-900">批次列表</h3>
                                <span className="text-sm text-gray-500">按创建时间排序</span>
                            </div>
                            <div className="overflow-x-auto">
                                <table className="min-w-full divide-y divide-gray-200">
                                    <thead className="bg-gray-50">
                                        <tr>
                                            <th scope="col" className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider w-64">批次信息</th>
                                            <th scope="col" className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">面额/门槛</th>
                                            <th scope="col" className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider w-48">库存情况</th>
                                            <th scope="col" className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">有效期/叠加</th>
                                            <th scope="col" className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">状态</th>
                                            <th scope="col" className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">操作更新/操作人</th>
                                            <th scope="col" className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">审核时间/审核人</th>
                                            <th scope="col" className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">审核备注</th>
                                            <th scope="col" className="px-6 py-3 text-right text-xs font-medium text-gray-500 uppercase tracking-wider sticky right-0 bg-gray-50">操作</th>
                                        </tr>
                                    </thead>
                                    <tbody className="bg-white divide-y divide-gray-200">
                                        {sortedCoupons.map((coupon) => {
                                            const remaining = coupon.totalQuantity - coupon.claimedQuantity;
                                            const isLowStock = remaining <= 50 && remaining > 0;
                                            const isSoldOut = remaining <= 0;
                                            
                                            return (
                                            <tr key={coupon.id} className="hover:bg-gray-50 transition-colors">
                                                <td className="px-6 py-4">
                                                    <div className="flex flex-col items-start gap-1">
                                                        <div className="flex items-center text-xs text-gray-400 font-mono mb-0.5">
                                                            <Tag size={12} className="mr-1" />
                                                            批次号: {coupon.batchNo || '-'}
                                                        </div>
                                                        <span className="text-sm font-medium text-gray-900">{coupon.name}</span>
                                                        <span className="text-xs text-blue-700 bg-blue-50 border border-blue-200 px-2 py-1 rounded">
                                                            {getScenarioLabel(coupon.scenario)}
                                                        </span>
                                                    </div>
                                                </td>
                                                <td className="px-6 py-4 whitespace-nowrap">
                                                    <div className="inline-block border border-gray-200 rounded-md px-3 py-1.5 bg-white shadow-sm">
                                                        <div className="text-sm text-gray-900 font-bold text-center">¥{coupon.amount}</div>
                                                        <div className="text-xs text-gray-500 text-center mt-0.5 border-t border-gray-100 pt-0.5">
                                                            {coupon.ruleType === 'none' ? '无门槛' : `满 ¥${coupon.thresholdAmount}`}
                                                        </div>
                                                    </div>
                                                </td>
                                                {/* 库存列 */}
                                                <td className="px-6 py-4 whitespace-nowrap">
                                                    <div className="text-sm text-gray-900">
                                                        <span className="text-gray-500">总量: </span>{coupon.totalQuantity}
                                                    </div>
                                                    
                                                    {/* 待审核库存提示 */}
                                                    {coupon.pendingStock !== null && (
                                                        <div className="text-xs text-orange-600 bg-orange-50 border border-orange-200 px-2 py-1 rounded mt-1 flex items-center gap-1 w-fit">
                                                            <RefreshCw size={10} />
                                                            <span>库存审核中: {coupon.pendingStock}</span>
                                                        </div>
                                                    )}

                                                    {(coupon.status === 'approved' || coupon.status === 'finished') && (
                                                        <div className="text-xs mt-1 space-y-0.5">
                                                            <div className="text-gray-500">已领: {coupon.claimedQuantity}</div>
                                                            <div className={`font-medium flex items-center gap-1 ${isLowStock ? 'text-red-600 font-bold' : (isSoldOut ? 'text-gray-400' : 'text-gray-500')}`}>
                                                                {isSoldOut ? (
                                                                    <span className="bg-gray-100 text-gray-500 px-1 rounded border border-gray-200">已抢光</span>
                                                                ) : (
                                                                    <>剩余: {remaining}</>
                                                                )}
                                                                {isLowStock && <AlertTriangle size={12} />}
                                                            </div>
                                                        </div>
                                                    )}
                                                </td>
                                                <td className="px-6 py-4 whitespace-nowrap">
                                                    <div className="text-sm text-gray-900">
                                                        {coupon.validityType === 'fixed' ? (
                                                            <div className="flex flex-col text-xs">
                                                                <span>{coupon.validityStartDate} 至</span>
                                                                <span>{coupon.validityEndDate}</span>
                                                            </div>
                                                        ) : (
                                                            <span>{coupon.validityDays}天有效</span>
                                                        )}
                                                    </div>
                                                    <div className="text-xs text-gray-500 mt-1">
                                                        {coupon.stackable ? '可叠加' : '不可叠加'}
                                                    </div>
                                                </td>
                                                <td className="px-6 py-4 whitespace-nowrap">
                                                    {getStatusBadge(coupon.status)}
                                                </td>
                                                {/* 操作更新/操作人 */}
                                                <td className="px-6 py-4 whitespace-nowrap">
                                                    <div className="text-sm text-gray-900">{new Date(coupon.updateTime).toLocaleString()}</div>
                                                    <div className="text-xs text-gray-500">{coupon.operator}</div>
                                                </td>
                                                {/* 审核时间/审核人 */}
                                                <td className="px-6 py-4 whitespace-nowrap">
                                                    {coupon.auditTime ? (
                                                        <>
                                                            <div className="text-sm text-gray-900">{new Date(coupon.auditTime).toLocaleString()}</div>
                                                            <div className="text-xs text-gray-500">{coupon.auditor}</div>
                                                        </>
                                                    ) : (
                                                        <span className="text-gray-400 text-xs">-</span>
                                                    )}
                                                </td>
                                                {/* 审核备注 */}
                                                <td className="px-6 py-4">
                                                    {coupon.auditRemark ? (
                                                        <div className="text-xs text-gray-600 max-w-[150px] truncate" title={coupon.auditRemark}>
                                                            {coupon.auditRemark}
                                                        </div>
                                                    ) : (
                                                        <span className="text-gray-400 text-xs">-</span>
                                                    )}
                                                </td>
                                                <td className="px-6 py-4 whitespace-nowrap text-right text-sm font-medium sticky right-0 bg-white shadow-[-10px_0_10px_-10px_rgba(0,0,0,0.1)]">
                                                    <div className="flex justify-end gap-2 items-center">
                                                        {/* 编辑按钮 */}
                                                        {(coupon.status === 'draft' || coupon.status === 'rejected' || coupon.status === 'approved') && (
                                                            <button 
                                                                onClick={() => openFormModal(coupon)} 
                                                                className="text-indigo-600 hover:bg-indigo-50 border border-indigo-200 px-3 py-1 rounded text-xs transition-colors"
                                                            >
                                                                编辑
                                                            </button>
                                                        )}
                                                        
                                                        {/* 提交按钮 */}
                                                        {coupon.status === 'draft' && (
                                                            <button 
                                                                onClick={() => handleSubmitAudit(coupon.id)} 
                                                                className="text-blue-600 hover:bg-blue-50 border border-blue-200 px-3 py-1 rounded text-xs transition-colors"
                                                            >
                                                                提交
                                                            </button>
                                                        )}

                                                        {/* 审核按钮 */}
                                                        {(coupon.status === 'auditing' || coupon.pendingStock !== null) && (
                                                            <button 
                                                                onClick={() => openAuditModal(coupon)} 
                                                                className="text-orange-600 hover:bg-orange-50 border border-orange-200 px-3 py-1 rounded text-xs transition-colors"
                                                            >
                                                                {coupon.pendingStock !== null ? '审库存' : '审核'}
                                                            </button>
                                                        )}

                                                        {/* 结束活动 */}
                                                        {coupon.status === 'approved' && (
                                                            <button 
                                                                onClick={() => handleFinish(coupon.id)} 
                                                                className="text-red-600 hover:bg-red-50 border border-red-200 px-3 py-1 rounded text-xs transition-colors"
                                                            >
                                                                结束
                                                            </button>
                                                        )}

                                                        {/* 删除 */}
                                                        {(['draft', 'rejected'].includes(coupon.status)) && (
                                                            <button onClick={() => handleDelete(coupon.id)} className="text-gray-400 hover:text-red-600 hover:bg-gray-100 p-1.5 rounded transition-colors" title="删除">
                                                                <Trash2 size={16} />
                                                            </button>
                                                        )}
                                                    </div>
                                                </td>
                                            </tr>
                                        )})}
                                        {sortedCoupons.length === 0 && (
                                            <tr>
                                                <td colSpan="9" className="px-6 py-10 text-center text-gray-500">
                                                    暂无优惠券数据，请点击右上角新建。
                                                </td>
                                            </tr>
                                        )}
                                    </tbody>
                                </table>
                            </div>
                        </div>
                    </main>

                    {/* 新增/编辑 模态框 */}
                    {isModalOpen && (
                        <div className="fixed inset-0 z-10 overflow-y-auto" aria-labelledby="modal-title" role="dialog" aria-modal="true">
                            <div className="flex items-end justify-center min-h-screen pt-4 px-4 pb-20 text-center sm:block sm:p-0">
                                <div className="fixed inset-0 bg-gray-500 bg-opacity-75 transition-opacity" onClick={() => setIsModalOpen(false)}></div>
                                <span className="hidden sm:inline-block sm:align-middle sm:h-screen" aria-hidden="true">&#8203;</span>
                                <div className="inline-block align-bottom bg-white rounded-lg text-left overflow-hidden shadow-xl transform transition-all sm:my-8 sm:align-middle sm:max-w-lg w-full">
                                    <form onSubmit={handleSave}>
                                        <div className="bg-white px-4 pt-5 pb-4 sm:p-6 sm:pb-4">
                                            <div className="sm:flex sm:items-start">
                                                <div className="mt-3 text-center sm:mt-0 sm:text-left w-full">
                                                    <h3 className="text-lg leading-6 font-medium text-gray-900 mb-4" id="modal-title">
                                                        {formData.id ? (isApproved ? '调整库存（运行中）' : '编辑优惠券批次') : '新建优惠券批次'}
                                                    </h3>
                                                    
                                                    {isApproved && (
                                                        <div className="mb-4 p-3 bg-blue-50 text-blue-700 text-sm rounded border border-blue-100 flex items-start gap-2">
                                                            <div className="mt-0.5"><IconBase size={16}><circle cx="12" cy="12" r="10"></circle><line x1="12" y1="16" x2="12" y2="12"></line><line x1="12" y1="8" x2="12.01" y2="8"></line></IconBase></div>
                                                            <div>
                                                                <p className="font-bold">修改说明</p>
                                                                <p>当前活动已通过审核。您仅可修改库存数量。</p>
                                                                <p>修改后的库存需要再次审核，审核期间原活动正常进行。</p>
                                                            </div>
                                                        </div>
                                                    )}

                                                    <div className="grid gap-4">
                                                        {/* 批次名称 */}
                                                        <div>
                                                            <label className="block text-sm font-medium text-gray-700">批次名称</label>
                                                            <input 
                                                                required
                                                                type="text" 
                                                                disabled={isApproved}
                                                                className={`mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 sm:text-sm ${isApproved ? 'bg-gray-100 text-gray-500' : 'focus:ring-blue-500 focus:border-blue-500'}`}
                                                                value={formData.name}
                                                                onChange={e => setFormData({...formData, name: e.target.value})}
                                                                placeholder="如：双11大促通用券"
                                                            />
                                                        </div>

                                                        {/* 库存设置 (新增) */}
                                                        <div className={isApproved ? "p-3 bg-yellow-50 border border-yellow-200 rounded-md" : ""}>
                                                            <label className="block text-sm font-medium text-gray-700">
                                                                发放总量 (张)
                                                                {isApproved && <span className="ml-2 text-xs text-yellow-600 font-bold">* 可修改</span>}
                                                            </label>
                                                            <input 
                                                                required
                                                                type="number" 
                                                                min="1"
                                                                className="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm"
                                                                value={formData.totalQuantity}
                                                                onChange={e => setFormData({...formData, totalQuantity: Number(e.target.value)})}
                                                                placeholder="例如: 1000"
                                                            />
                                                        </div>

                                                        {/* 金额设置 */}
                                                        <div className={`border border-gray-200 rounded-md p-3 ${isApproved ? 'bg-gray-100 opacity-80' : 'bg-gray-50/50'}`}>
                                                            <label className="block text-sm font-medium text-gray-700">面额 (元)</label>
                                                            <div className="mt-1 relative rounded-md shadow-sm">
                                                                <div className="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                                                                    <span className="text-gray-500 sm:text-sm">¥</span>
                                                                </div>
                                                                <input
                                                                    required
                                                                    type="number"
                                                                    min="0"
                                                                    disabled={isApproved}
                                                                    className={`block w-full pl-7 pr-12 sm:text-sm border-gray-300 rounded-md py-2 ${isApproved ? 'bg-gray-100 text-gray-500' : 'bg-white focus:ring-blue-500 focus:border-blue-500'}`}
                                                                    value={formData.amount}
                                                                    onChange={e => setFormData({...formData, amount: Number(e.target.value)})}
                                                                />
                                                            </div>
                                                        </div>

                                                        {/* 场景 */}
                                                        <div className={`border border-gray-200 rounded-md p-3 ${isApproved ? 'bg-gray-100 opacity-80' : 'bg-gray-50/50'}`}>
                                                            <label className="block text-sm font-medium text-gray-700">应用场景</label>
                                                            <select
                                                                disabled={isApproved}
                                                                className={`mt-1 block w-full pl-3 pr-10 py-2 text-base border-gray-300 sm:text-sm rounded-md ${isApproved ? 'bg-gray-100 text-gray-500' : 'bg-white focus:ring-blue-500 focus:border-blue-500'}`}
                                                                value={formData.scenario}
                                                                onChange={e => setFormData({...formData, scenario: e.target.value})}
                                                            >
                                                                {SCENARIOS.map(s => <option key={s.value} value={s.value}>{s.label}</option>)}
                                                            </select>
                                                        </div>

                                                        {/* 有效期设置 */}
                                                        <div className={`border border-gray-200 rounded-md p-3 ${isApproved ? 'bg-gray-100 opacity-80' : 'bg-gray-50/50'}`}>
                                                            <label className="block text-sm font-medium text-gray-700 mb-2">有效期设置</label>
                                                            
                                                            <div className="flex gap-4 mb-3">
                                                                <label className="inline-flex items-center">
                                                                    <input 
                                                                        type="radio" 
                                                                        disabled={isApproved}
                                                                        className="form-radio text-blue-600"
                                                                        name="validityType"
                                                                        checked={formData.validityType === 'relative'}
                                                                        onChange={() => setFormData({...formData, validityType: 'relative'})}
                                                                    />
                                                                    <span className="ml-2 text-sm text-gray-700">领取后生效 (天数)</span>
                                                                </label>
                                                                <label className="inline-flex items-center">
                                                                    <input 
                                                                        type="radio" 
                                                                        disabled={isApproved}
                                                                        className="form-radio text-blue-600"
                                                                        name="validityType"
                                                                        checked={formData.validityType === 'fixed'}
                                                                        onChange={() => setFormData({...formData, validityType: 'fixed'})}
                                                                    />
                                                                    <span className="ml-2 text-sm text-gray-700">固定时间段</span>
                                                                </label>
                                                            </div>

                                                            {formData.validityType === 'relative' ? (
                                                                <div>
                                                                     <input 
                                                                        required
                                                                        type="number" 
                                                                        min="1"
                                                                        disabled={isApproved}
                                                                        className={`block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 sm:text-sm ${isApproved ? 'bg-gray-100 text-gray-500' : 'bg-white focus:ring-blue-500 focus:border-blue-500'}`}
                                                                        value={formData.validityDays}
                                                                        onChange={e => setFormData({...formData, validityDays: Number(e.target.value)})}
                                                                        placeholder="例如: 7"
                                                                     />
                                                                     <p className="mt-1 text-xs text-gray-500">
                                                                        用户领取当日起生效，至第N天23:59失效
                                                                     </p>
                                                                </div>
                                                            ) : (
                                                                <div>
                                                                    <div className="flex items-center gap-2">
                                                                        <input 
                                                                            type="date" 
                                                                            required
                                                                            disabled={isApproved}
                                                                            className={`block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 sm:text-sm ${isApproved ? 'bg-gray-100 text-gray-500' : 'bg-white focus:ring-blue-500 focus:border-blue-500'}`}
                                                                            value={formData.validityStartDate}
                                                                            onChange={e => setFormData({...formData, validityStartDate: e.target.value})}
                                                                        />
                                                                        <span className="text-gray-500">至</span>
                                                                        <input 
                                                                            type="date" 
                                                                            required
                                                                            disabled={isApproved}
                                                                            className={`block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 sm:text-sm ${isApproved ? 'bg-gray-100 text-gray-500' : 'bg-white focus:ring-blue-500 focus:border-blue-500'}`}
                                                                            value={formData.validityEndDate}
                                                                            onChange={e => setFormData({...formData, validityEndDate: e.target.value})}
                                                                        />
                                                                    </div>
                                                                    <p className="mt-1 text-xs text-gray-500">
                                                                        无论何时领取，有效期均在指定时间段内，过期不可用
                                                                    </p>
                                                                </div>
                                                            )}
                                                        </div>

                                                        {/* 叠加规则 */}
                                                        <div>
                                                            <label className="block text-sm font-medium text-gray-700 mb-1">叠加规则</label>
                                                            <div className="flex gap-4">
                                                                <label className="inline-flex items-center">
                                                                    <input 
                                                                        type="radio" 
                                                                        disabled={isApproved}
                                                                        className="form-radio text-blue-600"
                                                                        name="stackable"
                                                                        checked={formData.stackable === true}
                                                                        onChange={() => setFormData({...formData, stackable: true})}
                                                                    />
                                                                    <span className="ml-2 text-sm text-gray-700">可叠加</span>
                                                                </label>
                                                                <label className="inline-flex items-center">
                                                                    <input 
                                                                        type="radio" 
                                                                        disabled={isApproved}
                                                                        className="form-radio text-blue-600"
                                                                        name="stackable"
                                                                        checked={formData.stackable === false}
                                                                        onChange={() => setFormData({...formData, stackable: false})}
                                                                    />
                                                                    <span className="ml-2 text-sm text-gray-700">不可叠加</span>
                                                                </label>
                                                            </div>
                                                        </div>

                                                        {/* 使用规则 */}
                                                        <div>
                                                            <label className="block text-sm font-medium text-gray-700 mb-1">使用门槛</label>
                                                            <div className="space-y-2">
                                                                <div className="flex gap-4">
                                                                    <label className="inline-flex items-center">
                                                                        <input 
                                                                            type="radio" 
                                                                            disabled={isApproved}
                                                                            className="form-radio text-blue-600"
                                                                            name="ruleType"
                                                                            checked={formData.ruleType === 'none'}
                                                                            onChange={() => setFormData({...formData, ruleType: 'none', thresholdAmount: 0})}
                                                                        />
                                                                        <span className="ml-2 text-sm text-gray-700">无门槛</span>
                                                                    </label>
                                                                    <label className="inline-flex items-center">
                                                                        <input 
                                                                            type="radio" 
                                                                            disabled={isApproved}
                                                                            className="form-radio text-blue-600"
                                                                            name="ruleType"
                                                                            checked={formData.ruleType === 'threshold'}
                                                                            onChange={() => setFormData({...formData, ruleType: 'threshold'})}
                                                                        />
                                                                        <span className="ml-2 text-sm text-gray-700">满减</span>
                                                                    </label>
                                                                </div>
                                                                
                                                                {formData.ruleType === 'threshold' && (
                                                                    <div className="relative rounded-md shadow-sm w-1/2">
                                                                        <div className="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                                                                            <span className="text-gray-500 sm:text-xs">满¥</span>
                                                                        </div>
                                                                        <input
                                                                            required
                                                                            type="number"
                                                                            min="0.01"
                                                                            step="0.01"
                                                                            disabled={isApproved}
                                                                            placeholder="门槛金额"
                                                                            className={`focus:ring-blue-500 focus:border-blue-500 block w-full pl-8 sm:text-sm border-gray-300 rounded-md py-1.5 ${isApproved ? 'bg-gray-100 text-gray-500' : ''}`}
                                                                            value={formData.thresholdAmount}
                                                                            onChange={e => setFormData({...formData, thresholdAmount: Number(e.target.value)})}
                                                                        />
                                                                    </div>
                                                                )}
                                                            </div>
                                                        </div>

                                                    </div>
                                                </div>
                                            </div>
                                        </div>
                                        <div className="bg-gray-50 px-4 py-3 sm:px-6 sm:flex sm:flex-row-reverse">
                                            <button type="submit" className="w-full inline-flex justify-center rounded-md border border-transparent shadow-sm px-4 py-2 bg-blue-600 text-base font-medium text-white hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 sm:ml-3 sm:w-auto sm:text-sm">
                                                保存
                                            </button>
                                            <button type="button" onClick={() => setIsModalOpen(false)} className="mt-3 w-full inline-flex justify-center rounded-md border border-gray-300 shadow-sm px-4 py-2 bg-white text-base font-medium text-gray-700 hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 sm:mt-0 sm:ml-3 sm:w-auto sm:text-sm">
                                                取消
                                            </button>
                                        </div>
                                    </form>
                                </div>
                            </div>
                        </div>
                    )}

                    {/* 审核 模态框 */}
                    {isAuditModalOpen && (
                        <div className="fixed inset-0 z-20 overflow-y-auto">
                            <div className="flex items-center justify-center min-h-screen px-4 pt-4 pb-20 text-center sm:block sm:p-0">
                                <div className="fixed inset-0 bg-gray-500 bg-opacity-75" onClick={() => setIsAuditModalOpen(false)}></div>
                                <div className="inline-block align-bottom bg-white rounded-lg text-left overflow-hidden shadow-xl transform transition-all sm:my-8 sm:align-middle sm:max-w-md w-full">
                                    <div className="bg-white px-4 pt-5 pb-4 sm:p-6">
                                        <h3 className="text-lg font-medium text-gray-900 mb-4">审核操作</h3>
                                        
                                        {/* 如果是库存审核，显示详细对比 */}
                                        {coupons.find(c => c.id === auditData.id)?.pendingStock !== null && (
                                            <div className="mb-4 bg-orange-50 p-3 rounded border border-orange-100 text-sm">
                                                <p className="font-bold text-orange-800 mb-1">库存变更申请</p>
                                                <div className="flex justify-between items-center text-gray-600">
                                                    <span>当前总量: <span className="font-mono font-bold">{coupons.find(c => c.id === auditData.id)?.totalQuantity}</span></span>
                                                    <span>→</span>
                                                    <span>申请总量: <span className="font-mono font-bold text-blue-600">{coupons.find(c => c.id === auditData.id)?.pendingStock}</span></span>
                                                </div>
                                            </div>
                                        )}

                                        <div className="mb-4">
                                            <span className="block text-sm font-medium text-gray-700 mb-2">审核结果</span>
                                            <div className="flex gap-4">
                                                <button 
                                                    onClick={() => setAuditData({...auditData, passed: true})}
                                                    className={`flex-1 py-2 rounded-md border text-sm font-medium ${auditData.passed ? 'bg-green-50 border-green-500 text-green-700' : 'bg-white border-gray-300 text-gray-700'}`}
                                                >
                                                    通过
                                                </button>
                                                <button 
                                                    onClick={() => setAuditData({...auditData, passed: false})}
                                                    className={`flex-1 py-2 rounded-md border text-sm font-medium ${!auditData.passed ? 'bg-red-50 border-red-500 text-red-700' : 'bg-white border-gray-300 text-gray-700'}`}
                                                >
                                                    拒绝
                                                </button>
                                            </div>
                                        </div>

                                        <div>
                                            <label className="block text-sm font-medium text-gray-700 mb-1">
                                                审核备注 { !auditData.passed && <span className="text-red-500">*</span> }
                                            </label>
                                            <textarea
                                                rows="3"
                                                className="shadow-sm focus:ring-blue-500 focus:border-blue-500 block w-full sm:text-sm border-gray-300 rounded-md p-2 border"
                                                placeholder={auditData.passed ? "选填，通过备注" : "必填，请填写拒绝原因"}
                                                value={auditData.remark}
                                                onChange={e => setAuditData({...auditData, remark: e.target.value})}
                                            />
                                        </div>
                                    </div>
                                    <div className="bg-gray-50 px-4 py-3 sm:px-6 sm:flex sm:flex-row-reverse">
                                        <button 
                                            onClick={handleAuditAction}
                                            disabled={!auditData.passed && !auditData.remark.trim()}
                                            className={`w-full inline-flex justify-center rounded-md border border-transparent shadow-sm px-4 py-2 text-base font-medium text-white sm:ml-3 sm:w-auto sm:text-sm ${(!auditData.passed && !auditData.remark.trim()) ? 'bg-gray-300 cursor-not-allowed' : 'bg-blue-600 hover:bg-blue-700'}`}
                                        >
                                            确认提交
                                        </button>
                                        <button onClick={() => setIsAuditModalOpen(false)} className="mt-3 w-full inline-flex justify-center rounded-md border border-gray-300 shadow-sm px-4 py-2 bg-white text-base font-medium text-gray-700 hover:bg-gray-50 sm:mt-0 sm:ml-3 sm:w-auto sm:text-sm">
                                            取消
                                        </button>
                                    </div>
                                </div>
                            </div>
                        </div>
                    )}

                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
