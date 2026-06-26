# React/Tailwind/Lucide Component Snippets

This document provides high-end React component templates styling the file upload process for organization settings branding assets and payment voucher uploads.

---

## 1. Organization Settings Branding Asset Uploader
This component handles circular brand icons (1:1) and rectangular brand logos (16:9). It transitions seamlessly through all required states.

```tsx
import React, { useState, useRef } from "react";
import { UploadCloud, CheckCircle2, AlertCircle, RefreshCw, Trash2, Image } from "lucide-react";

interface BrandingUploaderProps {
  type: "icon" | "logo";
  onUploadStateChange: (uploading: boolean) => void;
  onUploadSuccess: (url: string) => void;
}

export const BrandingAssetUploader: React.FC<BrandingUploaderProps> = ({
  type,
  onUploadStateChange,
  onUploadSuccess,
}) => {
  const [state, setState] = useState<"idle" | "uploading" | "success" | "error">("idle");
  const [progress, setProgress] = useState(0);
  const [errorMessage, setErrorMessage] = useState("");
  const [previewUrl, setPreviewUrl] = useState<string | null>(null);
  const fileInputRef = useRef<HTMLInputElement>(null);

  const simulateUpload = (file: File) => {
    setState("uploading");
    onUploadStateChange(true);
    setProgress(0);

    // Mock progress animation
    const interval = setInterval(() => {
      setProgress((prev) => {
        if (prev >= 100) {
          clearInterval(interval);
          setState("success");
          onUploadStateChange(false);
          const mockUrl = URL.createObjectURL(file);
          setPreviewUrl(mockUrl);
          onUploadSuccess(mockUrl);
          return 100;
        }
        return prev + 10;
      });
    }, 150);
  };

  const handleFileChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;

    // Validate size (max 2MB for branding)
    if (file.size > 2 * 1024 * 1024) {
      setState("error");
      setErrorMessage("El archivo excede el tamaño límite de 2MB.");
      return;
    }

    simulateUpload(file);
  };

  const handleDragOver = (e: React.DragEvent) => {
    e.preventDefault();
  };

  const handleDrop = (e: React.DragEvent) => {
    e.preventDefault();
    const file = e.dataTransfer.files?.[0];
    if (!file) return;
    if (file.size > 2 * 1024 * 1024) {
      setState("error");
      setErrorMessage("El archivo excede el tamaño límite de 2MB.");
      return;
    }
    simulateUpload(file);
  };

  const handleRetry = () => {
    setState("idle");
    setProgress(0);
    setErrorMessage("");
    fileInputRef.current?.click();
  };

  const handleDelete = () => {
    setState("idle");
    setPreviewUrl(null);
    setProgress(0);
  };

  const isRound = type === "icon";

  return (
    <div className="flex flex-col items-center justify-center w-full">
      <input
        type="file"
        ref={fileInputRef}
        onChange={handleFileChange}
        accept="image/png, image/jpeg, image/svg+xml"
        className="hidden"
      />

      {/* IDLE / EMPTY STATE */}
      {state === "idle" && (
        <div
          onDragOver={handleDragOver}
          onDrop={handleDrop}
          onClick={() => fileInputRef.current?.click()}
          className={`group relative flex flex-col items-center justify-center border-2 border-dashed border-slate-700 hover:border-indigo-500 bg-slate-900/40 hover:bg-slate-900/60 cursor-pointer transition-all duration-300 ease-in-out p-6 text-center ${
            isRound ? "w-36 h-36 rounded-full" : "w-full h-44 rounded-2xl"
          }`}
        >
          <UploadCloud className="w-8 h-8 text-slate-400 group-hover:text-indigo-400 group-hover:scale-110 transition-transform duration-300" />
          <span className="mt-2 text-xs font-medium text-slate-300 group-hover:text-white">
            {isRound ? "Subir Icono" : "Subir Logotipo"}
          </span>
          <span className="text-[10px] text-slate-500 mt-1">PNG, JPG, SVG</span>
        </div>
      )}

      {/* UPLOADING STATE */}
      {state === "uploading" && (
        <div
          className={`flex flex-col items-center justify-center bg-slate-900 border border-slate-800 p-6 text-center relative ${
            isRound ? "w-36 h-36 rounded-full" : "w-full h-44 rounded-2xl"
          }`}
        >
          <div className="relative flex items-center justify-center">
            {/* Spinning background track */}
            <svg className="w-16 h-16 transform -rotate-90">
              <circle
                cx="32"
                cy="32"
                r="26"
                stroke="currentColor"
                strokeWidth="4"
                className="text-slate-800"
                fill="transparent"
              />
              <circle
                cx="32"
                cy="32"
                r="26"
                stroke="currentColor"
                strokeWidth="4"
                className="text-indigo-500 transition-all duration-300"
                strokeDasharray={2 * Math.PI * 26}
                strokeDashoffset={2 * Math.PI * 26 * (1 - progress / 100)}
                fill="transparent"
              />
            </svg>
            <span className="absolute text-xs font-semibold text-white">{progress}%</span>
          </div>
          <span className="mt-2 text-xs text-slate-400 animate-pulse">Subiendo...</span>
        </div>
      )}

      {/* SUCCESS STATE */}
      {state === "success" && previewUrl && (
        <div
          className={`group relative overflow-hidden border border-emerald-500/30 bg-emerald-500/5 transition-all duration-300 flex items-center justify-center ${
            isRound ? "w-36 h-36 rounded-full" : "w-full h-44 rounded-2xl"
          }`}
        >
          <img src={previewUrl} alt="Preview" className="w-full h-full object-cover" />
          
          {/* Green Check Badge */}
          <div className="absolute top-2 right-2 bg-emerald-500 text-slate-950 rounded-full p-0.5 shadow-lg">
            <CheckCircle2 className="w-4 h-4 fill-emerald-500 text-slate-950" />
          </div>

          {/* Hover Overlay with Delete */}
          <div className="absolute inset-0 bg-slate-950/70 opacity-0 group-hover:opacity-100 flex items-center justify-center gap-3 transition-opacity duration-200">
            <button
              onClick={() => fileInputRef.current?.click()}
              className="p-2 bg-white/10 hover:bg-white/20 rounded-full text-white transition-colors"
              title="Reemplazar"
            >
              <RefreshCw className="w-4 h-4" />
            </button>
            <button
              onClick={handleDelete}
              className="p-2 bg-rose-500/20 hover:bg-rose-500/30 text-rose-400 hover:text-rose-300 rounded-full transition-colors"
              title="Eliminar"
            >
              <Trash2 className="w-4 h-4" />
            </button>
          </div>
        </div>
      )}

      {/* ERROR STATE */}
      {state === "error" && (
        <div
          className={`flex flex-col items-center justify-center border border-rose-500/30 bg-rose-500/5 text-center p-4 relative ${
            isRound ? "w-36 h-36 rounded-full" : "w-full h-44 rounded-2xl"
          }`}
        >
          <AlertCircle className="w-8 h-8 text-rose-500 mb-1" />
          <span className="text-[10px] text-rose-300 font-medium px-2 truncate w-full">
            {errorMessage || "Error de subida"}
          </span>
          <button
            onClick={handleRetry}
            className="mt-2 text-xs font-bold text-rose-400 hover:text-rose-300 transition-colors flex items-center gap-1"
          >
            <RefreshCw className="w-3 h-3" /> Reintentar
          </button>
        </div>
      )}
    </div>
  );
};
```

---

## 2. Payment Voucher Document Uploader
This component supports uploading files with a detailed horizontal layout, including name, size, linear progress bar, error state, and check icon success indicator.

```tsx
import React, { useState, useRef } from "react";
import { UploadCloud, FileText, CheckCircle2, AlertCircle, RefreshCw, X, File, ShieldAlert } from "lucide-react";

interface PaymentVoucherUploaderProps {
  onUploadStateChange: (uploading: boolean) => void;
  onUploadSuccess: (fileData: { name: string; size: string; url: string }) => void;
}

export const PaymentVoucherUploader: React.FC<PaymentVoucherUploaderProps> = ({
  onUploadStateChange,
  onUploadSuccess,
}) => {
  const [state, setState] = useState<"idle" | "scanning" | "uploading" | "success" | "error">("idle");
  const [progress, setProgress] = useState(0);
  const [fileDetails, setFileDetails] = useState<{ name: string; size: string } | null>(null);
  const [errorMessage, setErrorMessage] = useState("");
  const fileInputRef = useRef<HTMLInputElement>(null);
  const uploadTimerRef = useRef<any>(null);

  const formatBytes = (bytes: number): string => {
    if (bytes === 0) return "0 Bytes";
    const k = 1024;
    const sizes = ["Bytes", "KB", "MB"];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + " " + sizes[i];
  };

  const simulateUpload = (file: File) => {
    setFileDetails({
      name: file.name,
      size: formatBytes(file.size),
    });
    
    // Step 1: Security Virus Scan
    setState("scanning");
    onUploadStateChange(true);
    setProgress(0);

    uploadTimerRef.current = setTimeout(() => {
      // Step 2: Live upload progress bar
      setState("uploading");
      const interval = setInterval(() => {
        setProgress((prev) => {
          if (prev >= 100) {
            clearInterval(interval);
            setState("success");
            onUploadStateChange(false);
            onUploadSuccess({
              name: file.name,
              size: formatBytes(file.size),
              url: URL.createObjectURL(file),
            });
            return 100;
          }
          return prev + 8;
        });
      }, 100);
      
      // Save interval ref to clear if canceled
      uploadTimerRef.current = interval;
    }, 1200); // 1.2s scanning phase
  };

  const handleFileChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;

    if (file.size > 10 * 1024 * 1024) {
      setState("error");
      setErrorMessage("El comprobante excede el tamaño máximo permitido de 10MB.");
      return;
    }
    simulateUpload(file);
  };

  const cancelUpload = () => {
    if (uploadTimerRef.current) {
      clearTimeout(uploadTimerRef.current);
      clearInterval(uploadTimerRef.current);
    }
    setState("idle");
    setProgress(0);
    setFileDetails(null);
    onUploadStateChange(false);
  };

  return (
    <div className="w-full">
      <input
        type="file"
        ref={fileInputRef}
        onChange={handleFileChange}
        accept=".pdf, image/png, image/jpeg"
        className="hidden"
      />

      {/* IDLE DROPZONE */}
      {state === "idle" && (
        <div
          onClick={() => fileInputRef.current?.click()}
          className="group flex flex-col items-center justify-center w-full h-48 border-2 border-dashed border-slate-800 hover:border-indigo-500 bg-slate-900/30 hover:bg-slate-900/50 rounded-2xl cursor-pointer transition-all duration-300 p-6 text-center"
        >
          <div className="p-4 bg-indigo-500/10 rounded-full group-hover:scale-110 transition-transform duration-300">
            <UploadCloud className="w-8 h-8 text-indigo-400" />
          </div>
          <p className="mt-4 text-sm font-semibold text-slate-200">
            Arrastra tu comprobante de pago o <span className="text-indigo-400">búscalo aquí</span>
          </p>
          <p className="text-xs text-slate-500 mt-2">
            Soporta PDF, PNG, JPG (Máx. 10MB)
          </p>
        </div>
      )}

      {/* SCANNING STATE */}
      {state === "scanning" && fileDetails && (
        <div className="w-full p-5 bg-slate-900/80 border border-slate-800 rounded-2xl flex flex-col gap-4">
          <div className="flex items-center gap-3">
            <div className="p-3 bg-indigo-500/10 rounded-lg animate-pulse text-indigo-400">
              <FileText className="w-6 h-6" />
            </div>
            <div className="flex-1 min-w-0">
              <p className="text-sm font-medium text-white truncate">{fileDetails.name}</p>
              <p className="text-xs text-slate-400">{fileDetails.size}</p>
            </div>
            <button onClick={cancelUpload} className="text-slate-400 hover:text-white transition-colors">
              <X className="w-5 h-5" />
            </button>
          </div>
          <div className="flex items-center justify-between text-xs text-slate-400">
            <span className="flex items-center gap-1.5 font-medium text-indigo-400">
              <span className="w-2 h-2 rounded-full bg-indigo-500 animate-ping" />
              Escaneando seguridad...
            </span>
          </div>
          <div className="h-1.5 w-full bg-slate-800 rounded-full overflow-hidden">
            <div className="h-full bg-indigo-500 animate-pulse w-full" />
          </div>
        </div>
      )}

      {/* UPLOADING STATE (ACTIVE PROGRESS BAR) */}
      {state === "uploading" && fileDetails && (
        <div className="w-full p-5 bg-slate-900/80 border border-slate-800 rounded-2xl flex flex-col gap-4">
          <div className="flex items-center gap-3">
            <div className="p-3 bg-indigo-500/10 rounded-lg text-indigo-400">
              <FileText className="w-6 h-6" />
            </div>
            <div className="flex-1 min-w-0">
              <p className="text-sm font-medium text-white truncate">{fileDetails.name}</p>
              <p className="text-xs text-slate-400">{fileDetails.size}</p>
            </div>
            <button onClick={cancelUpload} className="text-slate-400 hover:text-white transition-colors">
              <X className="w-5 h-5" />
            </button>
          </div>
          <div className="flex items-center justify-between text-xs text-slate-400">
            <span>Subiendo...</span>
            <span className="font-semibold text-white">{progress}%</span>
          </div>
          {/* Progress track */}
          <div className="h-1.5 w-full bg-slate-800 rounded-full overflow-hidden relative">
            {/* Shimmering filling bar */}
            <div
              style={{ width: `${progress}%` }}
              className="h-full bg-gradient-to-r from-purple-500 to-indigo-500 transition-all duration-150 ease-out relative"
            >
              <div className="absolute inset-0 bg-white/20 animate-shimmer" style={{ backgroundSize: '200% 100%' }} />
            </div>
          </div>
        </div>
      )}

      {/* SUCCESS STATE */}
      {state === "success" && fileDetails && (
        <div className="w-full p-5 bg-emerald-500/5 border border-emerald-500/30 rounded-2xl flex items-center justify-between gap-4">
          <div className="flex items-center gap-3 min-w-0">
            <div className="p-3 bg-emerald-500/10 rounded-lg text-emerald-400">
              <CheckCircle2 className="w-6 h-6" />
            </div>
            <div className="flex-1 min-w-0">
              <p className="text-sm font-medium text-white truncate">{fileDetails.name}</p>
              <p className="text-xs text-emerald-400/80">{fileDetails.size} • Completado</p>
            </div>
          </div>
          <button
            onClick={() => setState("idle")}
            className="px-3 py-1.5 bg-slate-800 hover:bg-slate-700 text-xs font-semibold text-white rounded-lg transition-colors shrink-0"
          >
            Cambiar
          </button>
        </div>
      )}

      {/* ERROR STATE */}
      {state === "error" && (
        <div className="w-full p-5 bg-rose-500/5 border border-rose-500/30 rounded-2xl flex flex-col gap-4">
          <div className="flex items-start gap-3">
            <div className="p-3 bg-rose-500/10 rounded-lg text-rose-400 shrink-0">
              <AlertCircle className="w-6 h-6" />
            </div>
            <div className="flex-1">
              <p className="text-sm font-semibold text-white">Fallo al subir el comprobante</p>
              <p className="text-xs text-rose-300/80 mt-1">{errorMessage}</p>
            </div>
          </div>
          <div className="flex gap-3 justify-end">
            <button
              onClick={() => setState("idle")}
              className="px-3 py-1.5 text-xs text-slate-400 hover:text-white transition-colors"
            >
              Cancelar
            </button>
            <button
              onClick={() => {
                setState("idle");
                setTimeout(() => fileInputRef.current?.click(), 100);
              }}
              className="px-3 py-1.5 bg-rose-500 hover:bg-rose-600 text-xs font-semibold text-white rounded-lg transition-colors flex items-center gap-1.5"
            >
              <RefreshCw className="w-3.5 h-3.5" /> Reintentar
            </button>
          </div>
        </div>
      )}
    </div>
  );
};
```

---

## 3. Form Save/Submit Buttons Integration
This primary submit component checks if there are ongoing uploads and automatically manages its state, disabling interactions and displaying premium load animations.

```tsx
import React from "react";
import { Loader2, ArrowRight } from "lucide-react";

interface SubmitButtonProps {
  label: string;
  isUploading: boolean;  // Hooked to uploading state
  isSubmitting: boolean; // Hooked to form API submission
  onClick?: () => void;
}

export const FormSubmitButton: React.FC<SubmitButtonProps> = ({
  label,
  isUploading,
  isSubmitting,
  onClick,
}) => {
  const isDisabled = isUploading || isSubmitting;

  return (
    <button
      type={onClick ? "button" : "submit"}
      disabled={isDisabled}
      onClick={onClick}
      className={`relative w-full h-11 px-5 rounded-xl text-sm font-semibold flex items-center justify-center gap-2 select-none overflow-hidden transition-all duration-300
        ${
          isDisabled
            ? "bg-slate-800 text-slate-500 cursor-not-allowed border border-slate-700/50"
            : "bg-indigo-600 hover:bg-indigo-500 text-white shadow-lg shadow-indigo-600/10 hover:shadow-indigo-500/20 active:scale-[0.98]"
        }`}
    >
      {/* Background uploading state shimmer */}
      {isUploading && (
        <div className="absolute inset-0 bg-slate-800/80 flex items-center justify-center gap-2">
          <Loader2 className="w-4 h-4 text-slate-400 animate-spin" />
          <span className="text-slate-400 animate-pulse font-medium">Subiendo archivos...</span>
        </div>
      )}

      {/* Form API Submitting state spinner */}
      {!isUploading && isSubmitting && (
        <>
          <Loader2 className="w-4 h-4 text-white animate-spin" />
          <span>Guardando cambios...</span>
        </>
      )}

      {/* Regular state */}
      {!isUploading && !isSubmitting && (
        <>
          <span>{label}</span>
          <ArrowRight className="w-4 h-4 transition-transform group-hover:translate-x-1" />
        </>
      )}
    </button>
  );
};
```
